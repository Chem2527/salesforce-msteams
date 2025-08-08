# Microsoft Teams Employee App - SFCI Containerization Summary

**Project**: msteams-employee-app  
**Repository**: https://github.com/sf-itom/msteams-employee-app  
**Date**: August 7, 2025  
**Team**: SF-ITOM  
**Engineer**: Saikrishna Kakumanu

---

## Executive Summary

Successfully containerized Microsoft Teams Employee App using Salesforce Container Infrastructure (SFCI) with automated CI/CD pipeline. Docker images are now building and pushing to Salesforce's internal registry through Strata automation.

**Status**: SFCI Containerization Complete  
**Pipeline**: Fully Functional  
**Docker Images**: Building Successfully  

---

## Project Objectives

1. **Primary Goal**: Deploy Microsoft Teams Employee App to Salesforce Container Service (SCS)
2. **Technical Requirements**: 
   - mTLS support with certificate paths: `/etc/identity/ca`, `/etc/identity/server`, `/etc/identity/client`
   - Docker containerization
   - Automated CI/CD pipeline
   - Follow reference repository pattern: `service-agentforce-messaging`

---

## Implementation Timeline

### Phase 1: Initial Setup
- **Setup Strata CI/CD**: Created `.strata.yml` configuration
- **Configure Repository**: Set up GitHub integration with Strata
- **Pipeline Creation**: Established automated build triggers

### Phase 2: Docker Build Resolution
- **Issue**: `tsc: not found` during Docker build
- **Solution**: Changed `npm ci --only=production` to `npm install` to include dev dependencies
- **Result**: TypeScript compilation working

### Phase 3: Dependency Management
- **Issue**: Package lock file sync errors with `npm ci`
- **Solution**: Switched to `npm install` for flexibility
- **Enhancement**: Added `npm prune --production` for lean final image

### Phase 4: SSL Certificate Handling
- **Issue**: Missing localhost certificates in Docker environment
- **Solution**: Modified `vite.config.ts` to conditionally load certificates
- **Code Change**:
  ```typescript
  const useHttps = fs.existsSync(certPath) && fs.existsSync(keyPath);
  ```

### Phase 5: Registry Configuration
- **Target Registry**: `docker.repo.local.sfdc.net/sfci`
- **Image Name**: `msteams/msteams-employee-app-demo`
- **Authentication**: Integrated with Salesforce internal systems

### Phase 6: Strata Pipeline Optimization
- **Nexus Authentication**: Configured npm registry access
- **Build Commands**: Simplified to essential steps
- **Postinstall Scripts**: Added `--ignore-scripts` flag to prevent authentication issues

### Phase 7: Falcon Configuration Attempt
- **Manual BOM Creation**: Created `bom/service.json` and `bom/team.json`
- **Team Registration**: Attempted multiple team names (platform-engineering, chatbots, sf-itom, ITSM Vayu)
- **Result**: All teams returned 404 - not registered with Falcon

### Phase 8: SFCI Isolation Testing
- **Action**: Temporarily removed Falcon configuration
- **Purpose**: Test SFCI functionality independently
- **Result**: Complete success - all pipeline stages passed

---

## Technical Architecture

### Docker Configuration
```dockerfile
# Multi-stage build
FROM docker.repo.local.sfdc.net/sfci/docker-images/sfdc_rhel9_nodejs20 AS frontend-builder
FROM docker.repo.local.sfdc.net/sfci/docker-images/sfdc_rhel9_golang AS stage-2

# Runtime configuration
USER sfdc (UID: 7447)
WORKDIR /app
EXPOSE 8080
```

### Strata Pipeline Stages
1. **Setup** (8s): Environment preparation
2. **Precheck** (20s): Security and compliance checks
3. **Build** (1m 19s): npm build and Go tools compilation
4. **Package** (2m 52s): Docker image build and registry push
5. **Integration Test** (3m 50s): Container testing and validation

### Key Configuration Files
- `.strata.yml`: Pipeline definition
- `.strata.properties`: Global configuration
- `.strata-triggers.yml`: Trigger conditions
- `Dockerfile`: Container build instructions
- `docker-entrypoint.sh`: SCS deployment script

---

## Results Achieved

### Successful Outcomes
1. **Docker Images Built**: Successfully containerized application
2. **Registry Push**: Images available at `docker.repo.local.sfdc.net/sfci/msteams/msteams-employee-app-demo`
3. **Automated Pipeline**: Full CI/CD automation through Strata
4. **Integration Tests**: All tests passing
5. **mTLS Ready**: Certificate mount paths configured

### Pipeline Performance
- **Average Build Time**: ~7 minutes total
- **Success Rate**: 100% after configuration optimization
- **Artifact Size**: Optimized multi-stage build
- **Test Coverage**: Integration tests included

---

## Lessons Learned

### SFCI vs Falcon Distinction
- **SFCI**: Docker containerization and registry management (Complete)
- **Falcon**: Deployment platform requiring formal approvals (Future phase)

### Manual vs Automated Process
- **Manual BOM Creation**: Incorrect approach, caused validation failures
- **Proper Process**: Falcon service creation auto-generates required files
- **Approval Workflow**: FD (Falcon Deploy), FI (Falcon Infrastructure), CELL approvals required

### Authentication Challenges
- **Registry Access**: Requires Salesforce VPN and proper credentials
- **Nexus Integration**: Automated through Strata configuration
- **Team Registration**: Must be done through official Falcon onboarding

---

## Current Status

### Completed
- **SFCI Containerization**: Fully operational
- **Docker Build Pipeline**: Automated and successful
- **Registry Integration**: Images pushing successfully
- **CI/CD Automation**: Strata pipeline functional
- **Local Testing**: Docker images buildable locally

### Pending (Future Phase)
- **Falcon Service Creation**: Official request needed
- **Team Registration**: Requires DevOps/Falcon admin involvement
- **Deployment Approvals**: FD, FI, CELL approval workflow
- **Production Deployment**: Main branch deployment strategy

---

## Recommendations

### Immediate Actions
1. **Continue with SFCI**: Current setup is production-ready for containerization
2. **Falcon Service Request**: Initiate official Falcon onboarding process
3. **Team Registration**: Contact DevOps for proper team setup

### Technical Debt
1. **Certificate Management**: Implement proper SSL certificate handling for production
2. **Environment Configuration**: Set up proper dev/test/prod environment separation
3. **Monitoring Integration**: Add observability and logging

### Process Improvements
1. **Documentation**: Maintain updated deployment procedures
2. **Automation**: Extend CI/CD to include additional testing phases
3. **Security**: Regular security scanning and compliance checks

---

## Repository Structure

```
msteams-employee-app/
├── .strata.yml                 # CI/CD pipeline configuration
├── .strata.properties          # Global Strata settings
├── .strata-triggers.yml        # Build trigger configuration
├── Dockerfile                  # Container build instructions
├── docker-entrypoint.sh        # SCS deployment script
├── package.json                # Node.js dependencies
├── vite.config.ts              # Frontend build configuration
├── src/                        # Application source code
├── public/                     # Static assets
├── cmd/golang/                 # Go tools for SCS operations
└── temp-falcon-backup/         # Temporarily disabled Falcon config
    ├── falcon/                 # Falcon deployment configuration
    ├── bom/                    # Bill of Materials files
    └── CODEOWNERS              # Repository ownership
```

---

## Step-by-Step Progress Summary

**A → B → C Format:**

1. **Setup Strata** → **Configure .strata.yml** → **Pipeline Created**

2. **Docker Issues** → **Fix tsc/npm errors** → **Build Working**

3. **Registry Auth** → **Update registry URLs** → **SFCI Access**

4. **Nexus Auth** → **Fix npm authentication** → **Dependencies Resolved**

5. **Build Failures** → **Remove postinstall scripts** → **Clean Build**

6. **Add Falcon Config** → **Create BOM files manually** → **Validation Errors**

7. **Team Registration** → **Try multiple team names** → **All Teams 404**

8. **Remove Falcon** → **Test SFCI only** → **Pipeline SUCCESS**

9. **Local Testing** → **Build Docker locally** → **Image Created**

10. **Understanding** → **Learn Falcon vs SFCI** → **Process Clarified**

**Final Status:**
- SFCI Containerization: COMPLETE
- Docker Build/Push: WORKING 
- Strata Pipeline: SUCCESS
- Falcon Deployment: Needs Official Process

**Key Learning:**
Manual BOM → Should be auto-generated → Official approval needed

---

## Contact Information

**Primary Contact**: Saikrishna Kakumanu (saikrishna.kakumanu@salesforce.com)  
**Team**: SF-ITOM  
**Repository**: https://github.com/sf-itom/msteams-employee-app  
**Strata Dashboard**: [Internal Salesforce Strata Console]  

---

## Appendix

### Build Logs Summary
- **Latest Successful Build**: #16 (August 7, 2025)
- **Build Duration**: 7 minutes 38 seconds
- **Image Tag**: `jenkins-sf-itom-msteams-employee-app-feature-strata-github-integration-16-itest`
- **Registry**: `docker.repo.local.sfdc.net/sfci/msteams/msteams-employee-app-demo:latest`

### Key Metrics
- **Code Quality**: ESLint and TypeScript checks passing
- **Security**: No critical vulnerabilities detected
- **Performance**: Optimized multi-stage Docker build
- **Reliability**: 100% pipeline success rate after optimization

### Falcon Approval Process (For Future Reference)
- **FD (Falcon Deploy)**: Deployment system approval
- **FI (Falcon Infrastructure)**: Infrastructure capacity approval  
- **CELL (Falcon Cell)**: Regional placement approval
- **Process**: Official service request → Team registration → Approvals → Auto-generated configs

---

*Document Generated: August 7, 2025*  
*Version: 1.0*  
*Classification: Internal Salesforce Documentation*
