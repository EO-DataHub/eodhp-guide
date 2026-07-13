---
title: "C4 Detail: Identity & Access Management"
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - architecture
  - identity
---
# Detail: Identity & Access Management

Zooms into the Identity & Access Management container from the [main container diagram](containers.md), based on
the IAM architecture described in
[3.13 Identity and Access Management](../architectural-design/identity-and-access-management.md).

```kroki-plantuml
@startuml iam
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()

title EO Data Hub - Identity & Access Management (detail)

Person(user, "User", "API client or browser.")
System_Ext(idp, "Upstream Identity Provider", "GitHub, Google, Microsoft Entra ID.")
System_Ext(opaRepo, "OPA Policy Repo", "GitHub repo storing Rego policy as code.")

System_Boundary(iam, "Identity & Access Management") {
  Container(k8sproxy, "Kubernetes Proxy", "nginx", "Cluster ingress; forwards auth_request to Authagent.")
  Container(authagent, "Authagent", "Service", "Pre-authenticates and pre-authorizes requests using cookies, API tokens or Keycloak OAuth2 tokens.")
  Container(authagentOpa, "OPA (Authagent sidecar)", "Open Policy Agent", "Coarse-grained authorization sidecar for Authagent.")
  Container(oauth2proxy, "OAuth2 Proxy", "OAuth2 Proxy", "Handles browser-based OIDC login flow and session cookies (one instance per domain).")
  Container(keycloak, "Keycloak", "Keycloak", "Identity broker; issues and validates access tokens; OIDC client to upstream IdPs.")
  Container(opalServer, "OPAL Server", "OPAL", "Distributes system-managed authorization policy from the OPA repo.")
  Container(opalClient, "OPAL Client", "OPAL", "Subscribes to OPAL Server and pushes updates to local OPA sidecars.")
}

Container(eodhpService, "EODHP Service", "Any hub service", "Representative example of a hub service protected by IAM (e.g. Resource Catalogue, Web Presence).")
Container(serviceOpa, "OPA (service sidecar)", "Open Policy Agent", "Fine-grained authorization sidecar for the service.")
ContainerDb(userPolicies, "User Access Policies", "S3 (Workspace Storage)", "User-authored JSON policies ingested via the harvest pipeline.")

Rel(user, k8sproxy, "Sends API/UI request to", "HTTPS")
Rel(user, keycloak, "Logs in via", "OIDC")

Rel(k8sproxy, authagent, "Sends auth_request to")
Rel(k8sproxy, eodhpService, "Forwards request to (if authorized)")

Rel(authagent, authagentOpa, "Authorizes via")
Rel(authagent, oauth2proxy, "Validates cookie / redirects via")
Rel(oauth2proxy, keycloak, "Authenticates via", "OIDC")
Rel(keycloak, idp, "Federates via", "OIDC/SAML")

Rel(eodhpService, serviceOpa, "Authorizes via")
Rel(serviceOpa, opalClient, "Receives policy updates from")
Rel(authagentOpa, opalClient, "Receives policy updates from")
Rel(opalClient, opalServer, "Subscribes to")
Rel(opalServer, opaRepo, "Pulls system-managed policy from")

Rel(eodhpService, userPolicies, "Reads user-authored access policy from")

@enduml
```
