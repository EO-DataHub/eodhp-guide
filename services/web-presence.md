# Web Presence

## Summary

The web presence is the face of the EO DataHub and is accessible via the internet. Here users are able to sign up for an EO DataHub account and the web presence provides links to all aspects of the system together along with comprehensive documentation.

The web presence is also responsible for serving the Workspace UI static files

### Code Repositories and Artifacts

- Code available in https://github.com/EO-DataHub/eodhp-web-presence repository
- Workspace UI code available in https://github.com/EO-DataHub/eodhp-workspace-ui repository
- Container image published to public.ecr.aws/eodh/eodhp-web-presence AWS ECR
- Deployment is configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/web-presence directory

### Dependent Services

The web presence provides links to:
 - account sign-up
 - workspaces
 - catalogue UI
 - notebooks
 - information in the content management system (CMS)

The web presence provides a user interface for:
 - workspace management (including permissions and token generation)
 - account management (including billing)

Large parts of the system are available through an API so the web presence isn't always required for access. 

## Operation

The service runs as a Kubernetes deployment named `web-presence` under the `web` namespace.

### Configuration

The web presence is configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/web-presence directory.

The CMS can be updated at any time through the admin panel at https://eodatahub.org.uk/admin. Click Pages on the left hand side to be taken to a hierarchy of pages. Clicking on an individual page will allow the page to be edited.

Admin rights are required to edit the CMS, which can be done through the admin panel at https://eodatahub.org.uk/admin. Admin rights for individual users can be set in keycloak at https://eodatahub.org.uk/keycloak - the `hub_admin` role is required.


### Control

To restart service run `kubectl rollout restart -n web deployment web-presence` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

There is a postgres SQL database where data for the CMS is stored, along with a corresponding S3 bucket (web-artefacts-eodhp) for media files uploaded for the CMS. The `static-web-artefacts-eodh` S3 bucket contains static apps including the workspace UI and catalogue UI which are accessible from the web presence.

### Backups

All CMS data is in the eodhp postgres database with media files saved to the `web-artefacts-eodhp` bucket. These can be backed up using the database backup procedure if additional backups are required. This section details backing up for the purpose of exporting data.

Outputs are stored in the `web-database-exports-eodh` S3 bucket.

`python manage.py pg_dump` dumps the contents of the CMS, including database and media folders, to a timestamped folder in the exports bucket

`python manage.py pg_load timestamped_folder` loads the contents of the CMS, including database and media folders from the specified folder in the exports bucket


## Development

The web presence code is version controlled in the https://github.com/EO-DataHub/eodhp-web-presence repository.

New versions should be released by creating a new release using GitHub web UI with a version tag following the pattern v1.2.3. The commit tag will trigger the GitHub action release process.

Alternately, releases may be published directly from the code repository with `make publish version=v1.2.3`, but this should only be used for test releases as the Git commit will not be properly tagged.
