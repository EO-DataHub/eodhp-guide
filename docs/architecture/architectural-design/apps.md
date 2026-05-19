### 3.12 Apps

Apps are user-provided software that integrate with EODH in some fashion. An app provided by one user may act on behalf of another, it’s ‘end user’. 

#### 3.12.1 Stand-alone Tools as Apps

An App may be a software tool running on a user’s own computer or phone and accessing the hub using its APIs. The user must be an EODH user. The app can authenticate as its user either by asking the user to obtain and enter an EODH API key, or by integrating with Keycloak using established OIDC mechanisms (by launching a browser or using a flow for native apps). Integrations with Keycloak require the developer to contact an EODH administrator who must configure Keycloak manually with a new client. 

#### 3.12.2 Hub as Computational Back-End

In this scenario the application provider provides a service to users, for example server side SaaS, which authenticates itself to EODH using an API key. This is no different to any other client calling EODH APIs (typically workflow APIs in this case). The application provider is responsible for registering, authenticating and if necessary billing end-users, and end users may not be aware of EODH at all. Typically all access flows through the application provider's services, although some access may be direct, for example by providing pre-signed S3 URLs to end-users pointing to EODH object stores. 

#### 3.12.3 Application as Client Only

In this scenario the application is an OIDC client to EODH and may be a confidential or public client. The application provider must register as an OAuth client by asking the EODH operators to do this manually. End users log into the application using EODH, use their EODH identity within the application and are likely to need their own billing arrangement with EODH. 

This scenario could be used to read or write users’ data, to provide services on top of user provided data or to link EODH to other platforms. 

There's no mechanism for charging the end-user for use of the application and, if this is required, the application must do this independently. 

#### 3.12.4 Application as API via User Services

User services are specially configured workflows published by one workspace and callable by others. Inputs and outputs may come from the calling workspace but the workflow runs in the publishing workspace (and at the publishing workspace’s cost). The caller does not need to trust the publisher with access to their workspace and vice versa, but the workflow is still able to produce results by combining the caller’s inputs and the publisher’s proprietary data. 

A user service can be used as an API-based app in which the publisher performs access control and billing data collection in the app itself. Because the publishing workspace is billed for the workflow’s resource use in EODH, there is no requirement that the end user has an active billing account and the application provider is free to choose any pricing model they wish. 

#### 3.12.5 Application as Client and User Service

This combines the previous three scenarios: the application is an OIDC client which calls a user service via the EODH workflow APIs in order to use it as a computational back-end. 

The application must be registered as an OIDC client by an administrator and calls hub APIs on the end user’s behalf. The application then uses this to call a user service published specifically to serve as a computational back-end to the app. 

This allows a client-side application to use the hub as a computational back-end without any need for its end user to have a billing account in EODH and whilst controlling completely how the end user is charged. 

Future work could allow user service calls to be charged-for at prices set by their publisher, with the cost being added to the end user’s hub account and a credit being added to the user service publisher’s account. This would relieve the application provider of running their own billing infrastructure. Future work could also enhance the permissions model for user services so that they could be limited to only customers of the app, relieving the user service of having to implement its own access control. 

