---
title: "5. OPEN-SOURCE SOFTWARE REUSE"
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
tags:
  - kubernetes
---
## 5. OPEN-SOURCE SOFTWARE REUSE

This section identifies the most important open-source software used in the system and their respective licences.

| Component   Layer  | Software  | License |
| :---- | :---- | :---- |
| DAS: Data Access  Services | TiTiler  | MIT (Open source) |
| ENS: Event and  Notification   Service | Argo Events  | Apache 2.0 (Open source) |
| IAM: Identity and  Access   Management  | Keycloak  OAuth2 Proxy  Nginx | Apache 2.0 (Open source)  MIT (Open source)  BSD (Open source) |
| Messaging   System | Apache Pulsar  | Apache 2.0 |
| RC: Resource   Catalog | Stac-fastapi  Stac-fastapi-elasticsearch Elasticsearch | MIT (Open source)  MIT (Open source)  Elastic Licence \+SSPL  (not OSI approved but permissive) |
| Supporting  | ArgoCD  Elasticsearch  Logstash  Kibana  Prometheus  Kubernetes | Apache 2.0 (Open source)  Elastic Licence \+ SSPL (not OSI  approved but permissive)  Elastic Licence  Elastic Licence \+ SSPL  Apache 2.0 (Open source)  Apache 2.0 (Open source) |
| WAS: Workflow  and Analysis   System | JupyterHub/JupyterLab  | BSD (Open source) |
| Web Presence  | Wagtail CMS  Django  React | BSD (Open source)  BSD (Open source)  MIT (Open source) |
| WR: Workflow   Runner | EOEPCA ADES  Calrissian | Apache 2.0 (Open Source)  MIT |
