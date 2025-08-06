# SCS Certificate Setup and Environment Configuration

This guide covers setting up OpenSSL, generating self-signed certificates, and understanding SCS (Static Content Service) environment endpoints for development and testing purposes.

## 📋 Prerequisites

### Windows Package Manager Setup

Install required tools using Chocolatey:

```bash
# Install OpenSSL
choco install openssl

# Install kubectl (Kubernetes CLI)
choco install kubectl
```

### Alternative OpenSSL Installation

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

# Set environment variables
export MY_USER_CERT="certificates/client.pem"
export MY_USER_KEY="certificates/client-key.pem"
export MY_USER_CACERTS="certificates/cacerts.pem"
export ENVIRONMENT_TYPE="dev"
export FALCON_INSTANCE="dev1-uswest2"
export SERVICE_NAME="agent-svc-messaging"

# Your upload commands here
./scsCopyBinary/scsCopy \
  --debug \
  --source="your-file.js" \
  --destination="agent-svc-messaging://path/to/file.js"
```

## ⚠️ Security Considerations

### For Development/Testing Only

- ❌ **Do NOT use these certificates in production**
- ❌ **Self-signed certificates are not trusted by SCS servers**
- ❌ **Keep private keys (`ca-key.pem`, `client-key.pem`) secure**

### For Production Use

- ✅ **Obtain certificates from your organization's Certificate Authority**
- ✅ **Use certificates specifically issued for SCS access**
- ✅ **Follow your organization's certificate management policies**
- ✅ **Ensure certificates have proper Subject Alternative Names (SAN)**

## 🐛 Common Issues

### Certificate Not Trusted Error

```json
{
  "error": "x509: certificate signed by unknown authority"
}
```

**Cause**: SCS server doesn't trust your self-signed CA  
**Solution**: Obtain production certificates from authorized CA

### Connection Refused

```json
{
  "error": "connection refused"
}
```

**Cause**: Network access or incorrect endpoint  
**Solution**: Verify network connectivity and environment variables

### Certificate Expired

```bash
# Check certificate validity
openssl x509 -in certificates/client.pem -dates -noout
```

**Solution**: Regenerate certificates with longer validity period

## 📚 Additional Resources

- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [Kubernetes Certificate Management](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)
- [TLS/SSL Best Practices](https://wiki.mozilla.org/Security/Server_Side_TLS)

## 🤝 Support

For production certificate requests and SCS access:
1. Contact your platform team
2. Follow your organization's certificate request process
3. Ensure proper authorization for environment access

---

**Note**: This guide is for development and testing purposes. Always follow your organization's security policies for production deployments.
