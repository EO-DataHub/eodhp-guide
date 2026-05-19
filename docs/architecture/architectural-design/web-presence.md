### 3.10 Web Presence

Many pieces of the web presence consist of UIs provided by other components, such as JupyterHub as described in 3.7 Jupyter, and we reuse existing interfaces as much as possible. In order to provide navigation, tie components into a cohesive site, provide static content and serve as a launch-point for client-side apps, an instance of the Wagtail CMS will be installed. 

Wagtail is a content management system that runs inside the Python web framework Django. A CMS is chosen because this enables platform administrators and user documentation writers to edit site content and settings, including non-technical users, whilst the underlying Django platform allows us to easily add additional Django apps to the server to provide any more custom interfaces that might be required in the future. 

The web presence uses the Keycloak UI, part of the IAM component, to provide login forms and registration forms. The web presence also has its own React-based custom user interfaces to manage billing accounts and workspaces – workspace creation, member management, credentials management and usage and billing information. This includes a form for requesting a billing account, resulting in a message to EODH support for approval. 

The web presence functionality is available to users without logging in except where there is a necessary reason for this. This is typically because a charge is incurred, private data is being accessed or because a contract with a commercial data provider is required (eg, for accurate quotations or data orders). 

Static JavaScript apps like the workspaces UI are deployed into S3 and served by CloudFront, with the CMS page loading the assets from there. This means that the CMS provides the header and footer whereas the client-side app renders into the centre. The apps are deployed to version-specific locations and the CMS configuration determines which version is loaded. By doing this, the deployment of the static apps is decoupled from the deployment of the CMS – no need to update the CMS container images, for example. 

The Catalogue Browser UI from SparkGeo follows a similar pattern, although its location is not versioned and it does not use the CMS-rendered header and footer: it is deployed by SparkGeo to an S3 location, served from the EODH domain name by CloudFront and referenced by the CMS. 

Finally, API documentation using the Swagger UI is available at /api/docs. An API docs service in Kubernetes gathers this documentation from the individual documentation endpoints of EODHP microservices, plus from files where these are not available, and merges them into a single OpenAPI description. This can then be used directly or via the Swagger UI. 

