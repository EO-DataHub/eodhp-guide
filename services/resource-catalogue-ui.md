# Resource Catalogue User Interface

## Summary

The Resource Catalogue User Interface (RC UI) is a React-based web application that provides users with the ability to discover, browse, visualize, and interact with geospatial data from STAC (SpatioTemporal Asset Catalog) collections. The application enables users to search for satellite imagery and other Earth observation data, view items on an interactive map, analyze data properties, and initiate data ordering and processing workflows.

### Code Repositories and Artifacts

- Code available in [this repository](https://github.com/EO-DataHub/eodhp-rc-ui)
- Built artifacts (static files) are deployed to S3 buckets (`{environment}-eodhrc-website-files` for development sites in SparkGeo AWS and `static-web-artefacts-eodh` for deployment in the EODH AWS) and served via CloudFront
- Deployment is configured using AWS CDK in the `iac/` directory of the repository
- GitHub Actions workflow builds and deploys the application to S3

### Dependent Services

The Resource Catalogue UI is a client-side application that users rely on for:

- Discovering and browsing STAC collections and items
- Visualizing geospatial data on an interactive map
- Searching for data by spatial extent, time range, and custom filters
- Viewing detailed metadata about collections and items
- Initiating data orders and purchases

If the RC UI is unavailable, users can still access the underlying APIs directly (STAC API, workspace API), but will lose the graphical interface for data discovery and visualization.

## Operation

The service runs as a static single-page application (SPA) hosted on AWS S3 and distributed via CloudFront CDN. There are multiple environments:

- Development: `develop.eodhrc.sparkgeo.dev`
- Demo: `demo.eodhrc.sparkgeo.dev`
- Production: `https://eodatahub.org.uk/`

Traffic reaches the service through:

1. DNS (Route 53) resolves the domain to CloudFront distribution
2. CloudFront serves static files from S3 bucket with caching
3. The SPA loads and makes API requests to backend services
4. All routes are handled by `index.html` for client-side routing

### Configuration

The application is configured through:

1. **Environment Variables** (set at build time):
   - `VITE_HUB_BASE_URL` - Base URL for the STAC catalogue API
   - `VITE_KEYCLOAK_URL` - Keycloak authentication server URL
   - `VITE_KEYCLOAK_REALM` - Keycloak realm name
   - `VITE_KEYCLOAK_CLIENT_ID` - Keycloak client ID
   - `VITE_TITILER_BASE_URL` - TiTiler service URL for data visualization
   - `VITE_QLR_API_URL` - QGIS layer file API
   - `VITE_BASE_PATH` - Base path for the application (optional)

2. **STAC Collection Configuration**:
   - Collection categorization defined in `src/constants/collectionCategorizations.ts`
   - Collection rendering configuration in `src/library/collectionRenderingConfig.ts`
   - JSON schema forms for query interfaces in `src/json-schema-forms/`

3. **AWS CDK Infrastructure**:
   - Configuration files in `iac/configs/` for different environments (base.py, demo.py, develop.py)
   - Stack definition in `iac/stacks/static_site.py`

### Control

**To deploy a new version:**

1. Changes pushed to the repository trigger GitHub Actions
2. GitHub Actions runs `yarn build` to create production bundle
3. Static files from `dist/` are copied to the S3 bucket for the target environment
4. CloudFront may need cache invalidation for immediate updates

**To manually deploy:**

```sh
cd iac
source .venv/bin/activate
cdk deploy {environment}-eodhrc-static-site --profile {aws-profile}
```

### Dependencies

The Resource Catalogue UI depends on the following external services:

- **STAC Catalogue API** - Provides access to collections, items, and search functionality
- **Keycloak** - Provides user authentication and authorization
- **TiTiler Service** - Provides dynamic raster tile generation for visualizing geospatial data on the map
- **QGIS layer API (QLR)** - Provides API to produce QGIS layer files
- **Workspace API** - Provides workspace management and user collection functionality
- **S3** - For storing and serving the static files, as well as accessing user data and thumbnails
- **CloudFront** - CDN for distributing the application globally
- **Route 53** - DNS service for domain resolution

### Backups

As a static site, the application itself has no state to back up. All application code is version controlled in GitHub. The deployment infrastructure is defined as code in the `iac/` directory.

User data (workspaces, collections, orders) is stored by backend services, not in the RC UI.

To recover from a complete loss:

1. Restore the repository from GitHub
2. Run `yarn install` and `yarn build` to rebuild the application
3. Re-deploy using CDK to recreate the AWS infrastructure
4. Update DNS records if necessary

## Development

The Resource Catalogue UI is a React + TypeScript application built with Vite.

**Code location:** <https://github.com/EO-DataHub/eodhp-rc-ui>

**Development setup:**

1. Install Node.js (version specified in `.nvmrc`, currently >= 21.7.1)
2. Install Yarn package manager
3. Clone the repository
4. Run `nvm install` (if using nvm)
5. Run `yarn install` to install dependencies
6. Copy `.env.sample` to `.env` and configure environment variables
7. Configure STAC collections as documented in `docs/collection-configuration.md`
8. Run `yarn dev` to start the development server at `http://127.0.0.1:5173`

**Development commands:**

- `yarn dev` - Start development server with hot reload
- `yarn build` - Build production bundle (TypeScript compilation + Vite build)
- `yarn lint` - Run TypeScript type checking and ESLint
- `yarn prettier` - Format code with Prettier
- `yarn test` - Run Vitest unit tests
- `yarn preview` - Preview production build locally

**Release process:**

1. Create a pull request to the target branch (develop, demo, or production)
2. After review and approval, merge the pull request
3. GitHub Actions automatically builds and deploys to the corresponding environment

**Infrastructure deployment:**

Infrastructure changes are deployed using AWS CDK:

1. Navigate to the `iac/` directory
2. Activate Python virtual environment: `source .venv/bin/activate`
3. Install dependencies: `pip install -r requirements.txt`
4. Configure AWS CLI: `aws configure sso` and `aws sso login --profile {profile}`
5. Synthesize CloudFormation template: `cdk synth {environment}-eodhrc-static-site --profile {profile}`
6. Deploy infrastructure: `cdk deploy {environment}-eodhrc-static-site --profile {profile}`

**Note:** First-time deployment may require:

- Bootstrapping CDK in the AWS account: `cdk bootstrap --profile {profile}`
- Adding NS records to parent DNS zone after Route 53 hosted zone creation
- Waiting for certificate validation to complete
