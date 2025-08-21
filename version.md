# BUILDINFO.json Version Extraction Fix

## 🔍 **Problem Description**

The `docker-entrypoint.sh` script was failing to extract version information because the required `BUILDINFO.json` file was not being created during the build process. This caused the SCS (Shared Content Store) copy operation to use empty version values, resulting in incorrect deployment paths.

## ❌ **What Was Happening Before**

### Issue in `docker-entrypoint.sh` (Lines 6-8):
```bash
VERSION=$(grep version /app/dist/BUILDINFO.json|cut -d\" -f4)
MAJOR=$(echo "${VERSION}"|cut -d. -f1)
MINOR=$(echo "${VERSION}"|cut -d. -f2)
```

**Problem:** The file `/app/dist/BUILDINFO.json` **did not exist** because:
1. `npm run build` only ran `tsc && vite build`
2. No step created the required `BUILDINFO.json` file
3. The `grep` command failed silently
4. `VERSION`, `MAJOR`, and `MINOR` variables were **empty**

### Result:
- SCS deployment paths were malformed: `SERVICE_NAME://HASH//filename` (missing version)
- Instead of proper paths like: `SERVICE_NAME://HASH/0.1/filename`

## ✅ **What We Fixed**

### 1. **Modified `package.json`**

**Before:**
```json
{
  "scripts": {
    "build": "tsc && vite build"
  }
}
```

**After:**
```json
{
  "scripts": {
    "build": "tsc && vite build && npm run create-buildinfo",
    "create-buildinfo": "node -e \"const pkg = require('./package.json'); const fs = require('fs'); fs.writeFileSync('dist/BUILDINFO.json', JSON.stringify({version: pkg.version}, null, 2));\""
  }
}
```

### 2. **No Changes to `docker-entrypoint.sh`**

**The `docker-entrypoint.sh` script remains exactly the same:**
```bash
#!/bin/bash

set -e  # Exit on any error

HASH=${OBFUSCATED_FI_NAME:-$(/app/fiHash)}
VERSION=$(grep version /app/dist/BUILDINFO.json|cut -d\" -f4)  # ← This line now works!
MAJOR=$(echo "${VERSION}"|cut -d. -f1)                         # ← Now gets "0"
MINOR=$(echo "${VERSION}"|cut -d. -f2)                         # ← Now gets "1"

# Export variables for use in subshells
export HASH MAJOR MINOR SERVICE_NAME

# Use find -exec to avoid fragile for loop over find output
find /app/dist -type f -exec bash -c '
    SOURCE="$1"
    /app/scsCopy --source "${SOURCE}" --destination "${SERVICE_NAME}://${HASH}/${MAJOR}.${MINOR}/${SOURCE#/app/dist/}" &
' _ {} \;
wait

# Make sure we have a manifest file
# TODO: This is a hack! We should not be looping over unknown files!
/app/createManifest --destination /tmp/manifest.json

# Copy the manifest file to the SCS server
/app/scsCopy --source /tmp/manifest.json --destination "${SERVICE_NAME}://${HASH}/manifest.json"
```

**Key Point:** We didn't need to modify the `docker-entrypoint.sh` script at all. The issue was that the file it was trying to read didn't exist.

## 🔧 **How the Fix Works**

### 1. **Build Process Now:**
```bash
npm run build
  ↓
tsc && vite build && npm run create-buildinfo
  ↓
Creates dist/BUILDINFO.json with content:
{
  "version": "0.1.0"
}
```

### 2. **Docker Container Execution:**
```bash
# docker-entrypoint.sh runs:
VERSION=$(grep version /app/dist/BUILDINFO.json|cut -d\" -f4)  # Gets: "0.1.0"
MAJOR=$(echo "${VERSION}"|cut -d. -f1)                         # Gets: "0"
MINOR=$(echo "${VERSION}"|cut -d. -f2)                         # Gets: "1"

# SCS paths are now correct:
# SERVICE_NAME://HASH/0.1/index.html
# SERVICE_NAME://HASH/0.1/assets/index-*.js
```

## ✅ **Verification Results**

### Local Testing:
```bash
# After running npm run build:
$ cat dist/BUILDINFO.json
{
  "version": "0.1.0"
}

# Version extraction test:
$ VERSION=$(grep version dist/BUILDINFO.json|cut -d\" -f4)
$ MAJOR=$(echo "${VERSION}"|cut -d. -f1)
$ MINOR=$(echo "${VERSION}"|cut -d. -f2)
$ echo "VERSION: $VERSION, MAJOR: $MAJOR, MINOR: $MINOR"
VERSION: 0.1.0, MAJOR: 0, MINOR: 1
```

### CI/CD Compatibility:
- ✅ `.strata.yml` already runs `npm run build` (line 39)
- ✅ GitHub workflow already runs `npm run build` (line 30)
- ✅ Both will now automatically create `BUILDINFO.json`

## 📁 **Files Changed**

1. **`package.json`** - Added BUILDINFO.json creation to build process
2. **`docker-entrypoint.sh`** - **NO CHANGES** (the existing script now works correctly)

## 🎯 **Impact**

### Before Fix:
- ❌ SCS paths: `SERVICE_NAME://HASH//filename`
- ❌ Version variables empty
- ❌ Potential deployment issues

### After Fix:
- ✅ SCS paths: `SERVICE_NAME://HASH/0.1/filename`
- ✅ Version variables populated correctly
- ✅ Proper version-based deployment paths

## 🚀 **Deployment**

This fix is ready for production deployment. The changes are minimal, backward-compatible, and solve the version extraction issue without modifying the core deployment logic in `docker-entrypoint.sh`.
