# Auth Agent

## Summary

Auth Agent ties together several different auth related components for the platform to provide a consolidated auth middleware to authenticate selected incoming requests.

It also provides auth endpoints for external services (such as AWS Lambdas) to authenticate their requests.

As well as tying together separate authentication and authorisation services, it provides opaque token authentication to the platform and publishes RESTful endpoints to manage user tokens.

### Code Repositories and Artifacts

- Microservice defined in https://github.com/EO-DataHub/eodhp-auth-agent repository
- Microservice container image published to public.ecr.aws/eodh/eodhp-auth-agent AWS ECR
- Deployment is configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/auth-agent directory

### Dependent Services

As Auth Agent is called on every request to eodatahub.org.uk and eodatahub-workspaces.org.uk it is central to the operation of the platform. It should be considered critical to the operation of the platform.

## Operation

The service runs as a Kubernetes deployment named `auth-agent` under the `auth-agent` namespace.

The Auth Agent is called by the Nginx auth_request functionality on all requests to eodatahub.org.uk and eodatahub-workspaces.org.uk, with the exception of authentication endpoints (Keycloak and Oauth2-Proxy instances). All requests into the Nginx reverse proxy for the domains specified are sent via subrequest through the Auth Agent. If Auth Agent determines that the request is authorised it will pass the request upstream for processing. Otherwise it will return a 401 (unauthenticated) or 403 (unauthorised) as required and the request will be rejected.

### Configuration

The Auth Agent is configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-auth-agent) in the apps/auth-agent directory.

### Control

To restart service run `kubectl rollout restart auth-agent -n auth-agent` for Kubernetes cluster.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- Oauth2 Proxy (platform and workspaces) - Auth Agent calls the oauth2-proxy-platform and oauth2-proxy-workspaces services to determine if a request is authenticated.
- Open Policy Agent - Auth Agent calls the opal-client to determine if a request is authorised.
- Database Operator - The Auth Agent token service relies on an SQL database, which is managed by the Postgres Operator. If the database is not available then the service will not start.

### Backups

All state for the Auth Agent is within the database. Restoring a previous state involves following the database restore procedure.

## Development

The Auth Agent service code is version controlled in https://github.com/EO-DataHub/eodhp-auth-agent repository.

New versions should be released by creating a new release using GitHub web UI with a version tag following the pattern v1.2.3. The commit tag will trigger the GitHub action release process.

Alternately, releases may be published directly from the code repository with `make publish version=v1.2.3`, but this should only be used for test releases as the Git commit will not be properly tagged.
