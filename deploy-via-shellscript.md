# MS Teams Employee App - Salesforce Agent Force Integration

A Microsoft Teams application that integrates with Salesforce Agent Force for employee services, ticket management, and chat communication. Deployed on Salesforce Falcon SCS (Static Content Service).

## Project Structure

```
msteams-employee-app/
├── api/                          # Azure Functions backend
│   ├── host.json
│   ├── local.settings.json
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── config.ts
│       └── functions/
│           └── getUserProfile.ts
│
├── src/                          # React frontend source
│   ├── index.tsx                 # App entry point
│   ├── index.css                 # Global styles
│   ├── components/               # React components
│   │   ├── App.tsx               # Main app component
│   │   ├── Context.tsx           # Teams context
│   │   ├── Privacy.tsx           # Privacy page
│   │   ├── TermsOfUse.tsx        # Terms page
│   │   ├── main-header/          # Header component
│   │   ├── main-menu/            # Navigation menu
│   │   ├── main-tab/             # Main tab container
│   │   ├── service-catalog/      # Service catalog
│   │   ├── tickets/              # Ticket management
│   │   └── common/               # Shared components
│   ├── common/                   # Shared utilities
│   │   ├── ApiConstants.ts
│   │   ├── Constants.ts
│   │   ├── RestApiEndpoints.ts
│   │   ├── config/               # Configuration
│   │   ├── context-api/          # Context providers
│   │   └── data/                 # Data definitions
│   ├── custom-hooks/             # React hooks
│   ├── service/                  # API services
│   ├── types/                    # TypeScript types
│   └── utils/                    # Utility functions
│
├── public/                       # Static assets
│   ├── index.html                # Development HTML
│   ├── tab.html                  # Teams tab entry point
│   ├── auth-start.html           # Auth flow start
│   ├── auth-end.html             # Auth flow end
│   ├── routes.json               # SPA routing config
│   ├── favicon.ico
│   └── projRes/                  # Salesforce resources
│
├── appPackage/                   # Teams app package
│   ├── manifest.json             # Teams app manifest
│   ├── color.png                 # App icon (color)
│   └── outline.png               # App icon (outline)
│
├── certificates/                 # SSL certificates
│   ├── client.pem                # Client cert
│   ├── client-key.pem            # Client key
│   └── cacerts.pem               # CA certificates
│
├── scsCopyBinary/                # SCS deployment tool
│   └── scsCopy                   # Binary for uploads
│
├── infra/                        # Infrastructure as Code
│   ├── azure.bicep               # Azure resources
│   └── azure.parameters.json     # Deployment parameters
│
├── env/                          # Environment configs
├── dist/                         # Build output (generated)
│
├── Configuration Files
├── .env.production               # Production environment vars
├── teams-manifest-falcon.json    # Falcon-specific manifest
├── vite.config.ts                # Vite build config
├── tsconfig.json                 # TypeScript config
├── package.json                  # Dependencies
├── jest.config.ts                # Test configuration
├── eslint.config.js              # Linting rules
│
├── Deployment Scripts
├── upload-prod.ps1               # Production deployment
├── upload-stage.ps1              # Staging deployment
├── deploy-and-verify.ps1         # Deploy with verification
├── create-spa-files.ps1          # SPA file generation
└── Dockerfile                    # Container definition
```

## Quick Start

### Prerequisites
- Node.js 18+ and npm
- PowerShell (Windows)
- Access to Salesforce Falcon SCS
- Valid SSL certificates in certificates/ folder

### 1. Install Dependencies
```bash
npm install
```

### 2. Configure Environment
Create .env.production:
```bash
VITE_CLIENT_ID=f60c703b-b6a5-4e73-94ae-1ed512b1a6d3
VITE_START_LOGIN_PAGE_URL=https://cdn.scs.static.lightning.force.com/agent-svc-messaging/auth-start.html
VITE_FUNC_ENDPOINT=http://localhost:7071
VITE_FUNC_NAME=getUserProfile
VITE_TEAMS_APP_ID=sf-employee-app-prod
```

### 3. Development
```bash
# Start basic dev server
npm start
# Runs on https://localhost:53000/agent-svc-messaging/

# Start with Teams framework (recommended)
npm run dev:teamsfx
# Runs both frontend and backend with Teams integration

# Preview production build
npm run serve
```

### 4. Build and Deploy
```powershell
# Production deployment
.\upload-prod.ps1

# Staging deployment  
.\upload-stage.ps1
```

## Key Configuration Files

### routes.json - SPA Routing (Critical!)
```json
{
  "routes": [
    {"route": "/auth-callback", "serve": "/index.html"},
    {"route": "/", "serve": "/index.html"}
  ],
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/favicon.ico", "/assets/*", "/*.css", "/*.js"]
  }
}
```

### teams-manifest-falcon.json - Teams Integration
```json
{
  "id": "sf-employee-app-prod",
  "staticTabs": [
    {
      "entityId": "index",
      "name": "Home", 
      "contentUrl": "https://cdn.scs.static.lightning.force.com/agent-svc-messaging/tab.html"
    }
  ],
  "webApplicationInfo": {
    "id": "f60c703b-b6a5-4e73-94ae-1ed512b1a6d3"
  }
}
```

### vite.config.ts - Build Configuration
```typescript
export default defineConfig({
  plugins: [react(), nodePolyfills(), federation(...)],
  base: '/agent-svc-messaging/',  // CDN path
  build: {
    outDir: 'dist',
    copyPublicDir: true,          // Include routes.json
  },
  publicDir: 'public',
})
```

## Deployment Process

### Falcon SCS Deployment Flow
1. Build React App: npm run build creates dist/ folder
2. Upload Assets: Uses scsCopy binary to upload to S3
3. CDN Distribution: Files served via cdn.scs.static.lightning.force.com
4. Service Path: /agent-svc-messaging/ (your service identifier)

### Upload Script Details (upload-prod.ps1)
- Sets mTLS certificates for secure upload
- Uploads all dist/ contents recursively
- Maintains directory structure on CDN
- Includes critical routes.json for SPA routing

## Access URLs

After successful deployment:

- Main App: https://cdn.scs.static.lightning.force.com/agent-svc-messaging/index.html
- Teams Tab: https://cdn.scs.static.lightning.force.com/agent-svc-messaging/index.html#/tab
- Tab Entry: https://cdn.scs.static.lightning.force.com/agent-svc-messaging/tab.html
- Auth Flow: https://cdn.scs.static.lightning.force.com/agent-svc-messaging/auth-start.html

## Troubleshooting

### Blank Pages Issue
**Problem**: URLs accessible but showing blank content

**Root Cause**: Missing routes.json file for SPA routing

**Solution**: 
1. Ensure routes.json exists in public/ folder
2. Verify it's included in dist/ after build
3. Check upload logs confirm routes.json uploaded

### Authentication Issues
**Problem**: Teams auth not working

**Solutions**:
- Verify VITE_CLIENT_ID matches Azure AD app registration
- Check Teams manifest has correct webApplicationInfo.id
- Ensure auth URLs point to correct CDN paths

### Build/Upload Failures
**Common Issues**:
- Missing certificates in certificates/ folder
- Incorrect environment variables
- Network/VPN issues with Falcon SCS

### Development Server Issues
**Problem**: npm run dev fails

**Solution**: Use correct commands:
- npm start (basic development)
- npm run dev:teamsfx (full Teams development)

## Architecture Overview

### Frontend Stack
- React 18 with TypeScript
- Fluent UI for Teams integration
- Vite for fast builds and federation
- React Router for client-side routing

### Backend Integration
- Azure Functions for API endpoints
- Salesforce APIs for data operations
- Microsoft Graph for Teams integration

### Deployment Infrastructure
- Falcon SCS (Salesforce Cloud Storage)
- S3 for asset storage
- CloudFront CDN for global distribution

## Development Workflow

### Local Development
```bash
# Start basic dev server
npm start
# Runs on https://localhost:53000/agent-svc-messaging/

# Start with Teams framework (full development)
npm run dev:teamsfx
# Runs both frontend and backend with Teams integration

# Preview production build
npm run serve
```

### Testing
```bash
# Run tests once
npm test

# Run tests in watch mode
npm run test:watch

# Lint code
npm run lint
```

### Production Build
```bash
# Build for production
npm run build

# Deploy to staging
.\upload-stage.ps1

# Deploy to production
.\upload-prod.ps1
```

## Available Scripts

From package.json:
- npm start - Basic Vite dev server
- npm run dev:teamsfx - Full Teams development
- npm run build - Production build
- npm test - Run tests once
- npm run test:watch - Run tests in watch mode
- npm run serve - Preview production build
- npm run lint - Lint code

## Security & Authentication

### Microsoft Teams SSO
- Uses Azure AD for authentication
- Teams context provides user identity
- Configured via teams-manifest-falcon.json

### Salesforce Integration
- mTLS certificates for secure API calls
- Environment-specific configurations
- Token-based authentication for Salesforce APIs

## Additional Resources

- [Falcon Paved Path Documentation](https://docs.internal.salesforce.com/falcon/paved-path/)
- [Teams App Development Guide](https://docs.microsoft.com/en-us/microsoftteams/platform/)
- [Agent Service Messaging Repository](https://git.soma.salesforce.com/chatbots/service-agentforce-messaging)

## Current Status

- Environment configured with .env.production
- Routes.json included for SPA routing (key fix for blank pages)
- Teams manifest updated with proper IDs
- Successfully deployed to Falcon SCS
- URLs accessible at cdn.scs.static.lightning.force.com/agent-svc-messaging/
- Development server running at https://localhost:53000/agent-svc-messaging/

---

**Last Updated**: December 2024  
**Service Name**: agent-svc-messaging  
**Environment**: Production (prod1-uswest2)  
**CDN**: cdn.scs.static.lightning.force.com
