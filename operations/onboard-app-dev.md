# Onboard an Application Developer

## Purpose

Insutructions for a UK EODataHub admin to onboard a new application developer so they can begin developing their app against the platform.

## When to Use

When a new application developer has been accepted to develop an application for the platform.

## Operation

The following instructions assume that application developer has already been approved to develop an application for the platform.

Application developer apps will likely require to authenticate users, using the platform as an IdP. To do so, they will require an OIDC client to be created for them in Keycloak. They may require more than one, depending on their application.

If they are developing a purely frontend application, they will require a "public" client. If they are developing a backend app, they will require a "confidential" client. Do not use a confidential client in a front end app as the client secret will not be secure.

### Create Required OIDC Client(s)

1. Log into Keycloak admin panel
2. Ensure you are in the correct realm (`eodhp`)
3. Click "Clients" in left sidebar
4. Create client
5. Name the client. The Client ID can either be a meaningful name or a random ID, or a mixture of both. Generally, we have been giving them meaningful names. Complete description as necessary.
6. Decide whether this is to be a "public" (frontend, generally) or "confidential" (backend, generally) app. Set Client authentication as required (on=confidential, off=public).
7. Authorization off
8. Select the Authentication flows permitted. Generally, Standard flow is required. Direct access grants may be useful for testing, but is generally not required. Service accounts may be required if the client is to act as a machine user (not tied to any individual hub user).
9. If the client will need to log users in, complete the Access settings with app URL config. Example settings:
   ```
   Root URL: https://appdomain.com
   Home URL: https://appdomain.com
   Valid redirect URIs: /auth/callback  # modify this to the callback path of the app
   Valid post logout refirect URIs: +
   Web origins: +
   ```
10. Save client
11. If a confidential client, in Client details, Credentials tab, copy the Client Secret
12. Provide Client ID (and Client Secret, if confidential client) to app developer for use with their app.
13. Application developer will now be able to integrate their OIDC client with the EO DataHub using the platform as IdP.

### Add App Domain to Keycloak Content-Security-Policy

The Keycloak Content-Security-Policy needs to be updated to allow the app OIDC client to use an iframe to log in to the platform. To add the domain to the Keycloak CSP:

1. In Keycloak realm, visit Realm settings from the left sidebar.
2. Under security defenses tab, add domain (including protocol) to the Content-Security-Policy string under `frame-ancestors`. Example: `frame-src 'self'; frame-ancestors 'self' https://*.eodatahub.org.uk;  object-src 'none';` For example, if the app developer app is hosted on `https://myapp.example.com` then the CSP becomes `frame-src 'self'; frame-ancestors 'self' https://*.eodatahub.org.uk https://myapp.example.com;  object-src 'none';`. To allow for local app development, you can add `http://127.0.0.1:*`. To allow all subdomains of a domain you can use `http://*.example.com`. The general form of the CSP is `frame-src 'self'; frame-ancestors 'self' {space delimited domain names, including protocol}; object-src 'none';`.

## Requirements

- Keycloak admin panel access
- Keycloak realm role "admin"
