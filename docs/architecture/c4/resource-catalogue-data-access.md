---
title: "C4 Detail: Resource Catalogue & Data Access"
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - architecture
  - catalogue
---
# Detail: Resource Catalogue & Data Access

Zooms into the Resource Catalogue & Data Access container from the [main container diagram](containers.md), based
on [3.1 Architecture Overview](../architectural-design/architecture-overview.md)'s Figure 3-2, which breaks this
subsystem down into the harvest/ingest pipeline plus the data-access services sitting alongside it.

The Harvesters → Transformers → Ingesters pipeline is message-driven (via the Messaging/Pulsar backbone shown in
the [main container diagram](containers.md)), not a direct call chain.

```kroki-plantuml
@startuml resource_catalogue_data_access
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()

title EO Data Hub - Resource Catalogue & Data Access (detail)

System_Ext(workflowRunner, "Workflow Runner", "Registers new workflow-produced catalogue resources.")
System_Ext(openProviders, "Open Data Providers", "Open/public data sources (e.g. open data catalogues).")
ContainerQueue_Ext(messaging, "Messaging", "Apache Pulsar", "Event backbone connecting harvesting/ingestion pipeline stages.")
System_Ext(catalogueUI, "Resource Catalogue UI", "Catalogue Browser — searches the catalogue and requests quotes/orders for commercial data.")
System_Ext(commercialProviders, "Commercial Data Providers", "e.g. Airbus, Planet, Open Cosmos.")
System_Ext(gis, "GIS", "Desktop GIS applications/plugin integrations (QGIS, ArcGIS).")
System_Ext(workspaceStorage, "Workspace Storage", "S3 + NFS/EFS. Owned by the Workspaces subsystem.")

System_Boundary(dataAccess, "Resource Catalogue & Data Access") {
  Container(harvesters, "Harvesters", "Service", "Obtains catalogue metadata from sources inside and outside EODH.")
  Container(transformers, "Transformers", "Service", "Transforms raw harvested catalogue data into the form required by catalogue services.")
  Container(ingesters, "Ingesters", "Service", "Takes transformed catalogue data and inserts it into the STAC API.")
  Container(search, "STAC API", "stac-fastapi", "Customised STAC API providing catalogue access, including user-namespaced workspace entries.")
  Container(dataAdaptors, "Data Adaptors (Commercial Data)", "Service", "Airbus/Planet/Open Cosmos adaptors: catalogue harvesting plus quotation and ordering integration with commercial providers.")
  ContainerDb(userPolicies, "User Policies", "Store", "User-set access policies allowing workspace data to be published. Internal only.")
  Container(titiler, "TiTiler", "TiTiler", "WM(T)S / XYZ tile service for visualising user and data-stream raster data.")
  Container(httpsDownload, "HTTPS Download", "AWS Lambda + nginx", "HTTPS-based access to files in workspace object/block stores.")
}

Lay_R(harvesters, transformers)
Lay_R(transformers, ingesters)
Lay_R(ingesters, search)
Lay_U(harvesters, messaging)
Lay_U(transformers, messaging)
Lay_D(harvesters, dataAdaptors)
Lay_R(userPolicies, dataAdaptors)
Lay_R(search, titiler)
Lay_R(titiler, httpsDownload)
Lay_L(openProviders, harvesters)
Lay_D(catalogueUI, dataAdaptors)

Rel(harvesters, messaging, "Publishes harvested metadata to")
Rel(messaging, transformers, "Delivers harvested metadata to")
Rel(transformers, messaging, "Publishes transformed metadata to")
Rel(messaging, ingesters, "Delivers transformed metadata to")
Rel(ingesters, search, "Inserts catalogue entries into")
Rel(ingesters, userPolicies, "Reads user access policies from")

Rel(harvesters, openProviders, "Harvests catalogues from")
Rel(harvesters, dataAdaptors, "Uses for commercial-source harvesting")
Rel(dataAdaptors, commercialProviders, "Retrieves data from / quotes / orders")

Rel(workflowRunner, harvesters, "Registers new catalogue resources via")

Rel(catalogueUI, search, "Searches", "STAC API/HTTPS")
Rel(catalogueUI, dataAdaptors, "Requests quotes and orders via")
Rel(catalogueUI, titiler, "Renders map previews via", "XYZ/WMTS")
Rel(gis, titiler, "Renders layers via", "XYZ/WMTS")

Rel(titiler, search, "Reads STAC metadata from")
Rel(titiler, workspaceStorage, "Reads raster data from")
Rel(httpsDownload, workspaceStorage, "Reads/writes files in")

@enduml
```
