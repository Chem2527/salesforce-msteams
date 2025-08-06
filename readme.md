# SCS Certificate Setup and Environment Configuration

This guide covers setting up OpenSSL, generating self-signed certificates, and understanding SCS (Static Content Service) environment endpoints for development and testing purposes.

## 📋 Prerequisites

### Windows Package Manager Setup

Install required tools using Chocolatey:

```bash

# Install kubectl (Kubernetes CLI)
choco install kubectl
```

###  OpenSSL Installation

If you prefer manual installation, download OpenSSL from:
- **Windows Binaries**: https://slproweb.com/products/Win32OpenSSL.html

## 🔐 Certificate Generation

### ⚠️ Important Warning

**The certificates generated below are self-signed and will NOT work with production SCS servers** because they are not trusted by the server's Certificate Authority. These are for **development and testing purposes only**.

### 1. Generate Certificate Authority (CA)

```bash
# Generate CA private key
openssl genrsa -out ca-key.pem 2048

# Generate CA certificate (valid for 365 days)
openssl req -new -x509 -days 365 -key ca-key.pem -out cacerts.pem -subj "/C=US/ST=CA/L=SanFrancisco/O=employee/CN=CA"
```

### 2. Generate Client Private Key

```bash
# Generate client private key (2048-bit RSA)
openssl genrsa -out client-key.pem 2048
```

### 3. Create Client Certificate Signing Request (CSR)

```bash
# Generate client certificate signing request
openssl req -new -key client-key.pem -out client.csr -subj "/C=US/ST=CA/L=SanFrancisco/O=employee/CN=client"
```

### 4. Sign Client Certificate with CA

```bash
# Sign the client certificate with your CA (valid for 365 days)
openssl x509 -req -in client.csr -CA cacerts.pem -CAkey ca-key.pem -CAcreateserial -out client.pem -days 365
```

### 5. Verify Generated Certificates

```bash
# Verify client certificate
openssl verify -CAfile cacerts.pem client.pem

# Check certificate details
openssl x509 -in client.pem -text -noout

# Check certificate dates
openssl x509 -in client.pem -dates -noout
```

## 📁 Generated Certificate Files

After completing the steps above, you should have these files:

```
certificates/
├── cacerts.pem      # Certificate Authority (CA) certificate
├── ca-key.pem       # CA private key (keep secure!)
├── client.pem       # Client certificate
├── client-key.pem   # Client private key
└── client.csr       # Certificate signing request (can be deleted)
```

## 🏗️ SCS Environment Configuration

### Environment Endpoint Mapping

| FALCON_INSTANCE | ENVIRONMENT_TYPE | Functional Domain | SCS Endpoint |
|-----------------|------------------|-------------------|--------------|
| `dev1-uswest2` | `dev` | `core002` | `scs-slb.sfproxy.core002.dev1-uswest2.aws.sfdc.cl:7443` |
| `test1-uswest2` | `test` | `core3` | `scs-slb.sfproxy.core3.test1-uswest2.aws.sfdc.cl:7443` |
| `aws-stage1-useast2` | `stage` | `core1` | `scs-slb.sfproxy.core1.aws-stage1-useast2.aws.sfdc.cl:7443` |
| `aws-prod0-uswest2` | `prod` | `core1` | `scs-slb.sfproxy.core1.aws-prod0-uswest2.aws.sfdc.cl:7443` |

### Environment Variables Setup

```bash
# Development Environment
export FALCON_INSTANCE="dev1-uswest2"
export ENVIRONMENT_TYPE="dev"
export SERVICE_NAME="agent-svc-messaging"

# Certificate Paths
export MY_USER_CERT="certificates/client.pem"
export MY_USER_KEY="certificates/client-key.pem"
export MY_USER_CACERTS="certificates/cacerts.pem"
```

### Testing Different Environments

#### Development
```bash
export FALCON_INSTANCE="dev1-uswest2"
export ENVIRONMENT_TYPE="dev"
# Connects to: scs-slb.sfproxy.core002.dev1-uswest2.aws.sfdc.cl:7443
```

#### Test
```bash
export FALCON_INSTANCE="test1-uswest2"
export ENVIRONMENT_TYPE="test"
# Connects to: scs-slb.sfproxy.core3.test1-uswest2.aws.sfdc.cl:7443
```

#### Staging
```bash
export FALCON_INSTANCE="aws-stage1-useast2"
export ENVIRONMENT_TYPE="stage"
# Connects to: scs-slb.sfproxy.core1.aws-stage1-useast2.aws.sfdc.cl:7443
```

#### Production
```bash
export FALCON_INSTANCE="aws-prod0-uswest2"
export ENVIRONMENT_TYPE="prod"
# Connects to: scs-slb.sfproxy.core1.aws-prod0-uswest2.aws.sfdc.cl:7443
```

## 🔧 Usage Example

### Sample Upload Script Configuration

```bash
#!/bin/bash

set -e

echo "🚀 Building React App"
npm run build

echo "📦 Uploading using scsCopy"

# Set environment variables for mTLS certificates once
export MY_USER_CERT=certificates/client.pem
export MY_USER_KEY=certificates/client-key.pem
export MY_USER_CACERTS=certificates/cacerts.pem
export ENVIRONMENT_TYPE=dev
export FALCON_INSTANCE="dev1-uswest2"
export SERVICE_NAME="agent-svc-messaging"

echo "📁 Uploading dist/ directory contents"
# Upload all files recursively inside dist
find dist -type f | while read -r file; do
  echo "  Uploading: $file"
  # Remove 'dist/' prefix for destination path to maintain structure
  dest_path=${file#dist/}
  ./scsCopyBinary/scsCopy \
    --debug \
    --source="$file" \
    --destination="agent-svc-messaging://$dest_path"
done

echo "🐳 Uploading Dockerfile"
# Check current directory for Dockerfile
if [ -f "./Dockerfile" ]; then
  echo "  Found Dockerfile in current directory"
  echo "  Uploading: ./Dockerfile"
  ./scsCopyBinary/scsCopy \
    --debug \
    --source="./Dockerfile" \
    --destination="agent-svc-messaging://Dockerfile"
else
  echo "❌ Error: Dockerfile not found in current directory"
  echo "📍 Current directory: $(pwd)"
  echo "📋 Available files:"
  ls -la | grep -E "(Dockerfile|dockerfile)" || echo "   No Dockerfile found"
  exit 1
fi

echo "✅ Upload completed successfully"
```


