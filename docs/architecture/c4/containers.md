---
title: C4 Containers
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - architecture
---
# Containers

The Hub's main deployable services. Two subsystems — Identity & Access Management, and Resource Catalogue & Data
Access — are shown collapsed here; see their detail pages for internals.

```kroki-plantuml
@startuml container
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()

title EO Data Hub - Container Diagram

Person(user, "User", "Expert science users, service developers, commercial users, government departments, etc.")

System_Ext(gis, "GIS", "Desktop GIS applications/plugin integrations (QGIS, ArcGIS).")
System_Ext(commercialProviders, "Commercial Data Providers", "e.g. Airbus, Planet, Open Cosmos.")
System_Ext(openProviders, "Open Data Providers", "Open/public data sources (e.g. open data catalogues).")
System_Ext(identityProvider, "Identity Provider", "External IdP (GitHub, Google, Microsoft Entra ID).")

System_Boundary(eodh, "EO Data Hub") {
  Container(webPresence, "Web Presence", "Wagtail CMS + Workspace UI", "Site content, billing and workspace management.")
  Container(catalogueUI, "Resource Catalogue UI", "React", "Catalogue Browser UI for STAC search, visualisation and ordering.")
  Container(dataAccess, "Resource Catalogue & Data Access", "stac-fastapi + TiTiler + Harvesters + Data Adaptors", "Discovers, harvests, catalogues and serves EO data: STAC search, tile rendering, and commercial/open data ingestion.")
  Container(jupyterhub, "JupyterHub", "JupyterHub/JupyterLab", "Launches and manages per-workspace notebook instances.")
  Container(workflowRunner, "Workflow Runner", "Workflow API + ADES/ZOO-Project", "Accepts, authorizes and executes CWL-based user workflows.")
  ContainerDb(workspaceStorage, "Workspace Storage", "S3 + NFS/EFS", "Object and block storage for workspaces and the catalogue.")
  Container(workspaceMgmt, "Workspace Management", "Python", "Creates and manages workspaces and their resources.")
  Container(accounting, "Accounting and Costing Services", "Python", "Tracks usage and billing.")
  ContainerQueue(messaging, "Messaging", "Apache Pulsar", "Event backbone connecting services.")
  Container(iam, "Identity & Access Management", "Keycloak + Authagent + OAuth2 Proxy + OPA", "Authenticates and authorizes requests to hub services.")
}

Lay_R(webPresence, catalogueUI)
Lay_R(catalogueUI, jupyterhub)
Lay_D(catalogueUI, dataAccess)
Lay_R(dataAccess, workflowRunner)
Lay_D(dataAccess, workspaceStorage)
Lay_D(webPresence, workspaceMgmt)
Lay_R(workspaceMgmt, accounting)
Lay_R(accounting, iam)
Lay_D(iam, messaging)
Lay_D(iam, identityProvider)

Rel(user, webPresence, "Manages account, billing and workspaces via", "HTTPS")
Rel(user, catalogueUI, "Searches, browses and orders EO data via", "HTTPS")
Rel(user, jupyterhub, "Runs notebooks via", "HTTPS")
Rel(user, gis, "Uses")

Rel(gis, dataAccess, "Searches catalogue and renders layers via", "STAC API/XYZ/WMTS")
Rel(gis, iam, "Authenticates via")

Rel(catalogueUI, dataAccess, "Searches", "STAC API/HTTPS")
Rel(webPresence, workspaceMgmt, "Creates and manages workspaces via")
Rel(webPresence, accounting, "Retrieves billing/usage info from")

Rel(dataAccess, openProviders, "Harvests catalogues from")
Rel(dataAccess, commercialProviders, "Retrieves data from (via Data Adaptors)")

Rel(dataAccess, workspaceStorage, "Reads raster data from")
Rel(workflowRunner, workspaceStorage, "Reads/writes workflow data")
Rel(workflowRunner, dataAccess, "Registers workflow outputs in")
Rel(jupyterhub, workspaceStorage, "Reads/writes user data")

Rel(dataAccess, messaging, "Publishes harvest/catalogue change events to")
Rel(accounting, messaging, "Consumes usage events from")

Rel(webPresence, iam, "Authenticates via")
Rel(catalogueUI, iam, "Authenticates via")
Rel(jupyterhub, iam, "Authenticates via")
Rel(iam, identityProvider, "Federates via", "OIDC")

note bottom of dataAccess
  Grouped subsystem — see the Resource Catalogue & Data Access
  detail page for the STAC API, TiTiler, HTTPS Download,
  Harvesters, Transformers, Ingesters and Data Adaptors internals.
end note

@enduml
```
