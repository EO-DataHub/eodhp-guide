---
title: C4 System Context
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - architecture
---
# System Context

The EO Data Hub, its users, and the external systems it integrates with.

```kroki-plantuml
@startuml system_context
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

LAYOUT_WITH_LEGEND()

title EO Data Hub - System Context

Person(user, "User", "Expert science users, service developers, commercial users, government departments, etc.")

System(eodh, "EO Data Hub", "Discovers, processes and provides access to Earth observation data.")

System_Ext(gis, "GIS", "Desktop GIS applications/plugin integrations (QGIS, ArcGIS).")
System_Ext(commercialProviders, "Commercial Data Providers", "Commercial data sources (e.g. Airbus, Planet, Open Cosmos).")
System_Ext(openProviders, "Open Data Providers", "Open/public data sources (e.g. open data catalogues).")
System_Ext(idp, "Identity Provider", "External IdP federated via Keycloak/OIDC.")

Rel(user, eodh, "Uses", "HTTPS")
Rel(user, gis, "Uses")

Rel(gis, eodh, "Uses", "API/HTTPS")

Rel(eodh, commercialProviders, "Retrieves data from")
Rel(eodh, openProviders, "Retrieves data from")
Rel(eodh, idp, "Authenticates users via", "OIDC")

@enduml
```
