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

The web presence primarilly relies on a Content Management System (CMS) called Wagtail, which can be configured using the admin panel for the website by an authenticated user. Other top-level pages can be configured within the web-presence code itself.

The web presence also uses a Helpscout Beacon to allow users to raise queries directly to Hub Admins. This Beacon is configured using the [Helpscout UI](https://secure.helpscout.net/settings/beacons/), where you can edit how the Beacon is displayed on the DataHub webpage, as well as alter the available options displayed when the Beacon is selected. Note, you need a Helpscout account to make changes to the Beacon using the UI, this is not the same as a Hub Admin account. The Beacon currently displayed on the Hub webpage is the `EODataHub Help` Beacon. The Beacon itself can be edited by selecting `Manage` in the navbar and then selecting `Beacons` in the dropdown. To edit the Problem Types displayed in the first page of the Beacon you need to edit the Custom Fields defined in the Helpscout client. Select `Inboxes` in the top left of the UI and select `EO Data Hub`, then select the Settings option (⚙️) in the bottom left of the UI and select `Custom Fields`, then select the Problem Type Field and make any changes before selecting `Save`.

### Configuration

The web presence is configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/web-presence directory.

The top-level pages that sit in front of the CMS can be configured in the [web-presence](https://github.com/EO-DataHub/eodhp-web-presence) repository, under eodhp_web_presence/core/templates. For example, to edit the nav bar and dropdown menus you can edit the menu.html file in the templates directory.

The CMS can be updated at any time through the admin panel at https://eodatahub.org.uk/admin. Admin rights for individual users can be set in keycloak at https://eodatahub.org.uk/keycloak - the `hub_admin` role is required. Click Pages on the left hand side to be taken to a hierarchy of pages. Clicking on an individual page will allow the page to be edited. 

#### Menu Items

Menu items (About, Data, Getting Started) are hardcoded in the template file and must be edited in code, not through the Wagtail CMS UI.

**Location:** https://github.com/EO-DataHub/eodhp-web-presence/blob/main/eodhp_web_presence/core/templates/menu.html

**To update menu items:**
- Edit the `menu.html` template file directly
- Modify the dropdown menu sections (About, Data, Getting Started) to change labels or URLs
- Add/remove menu items by copying the existing dropdown structure

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
