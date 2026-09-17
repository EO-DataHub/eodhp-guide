---
title: Elevate User
doc_status: ok
tags:
  - keycloak
  - identity
last_reviewed:
reviewed_by:
review_notes: "Copied from docs/operations/elevate-user.md"
---

# Elevate User

## Purpose

This operation is required when you want to assign a platform realm role to a user. See the [EODH realm roles](../../reference/iam/realm-roles.md) reference for the available roles and their intended use.

`admin` is intended for maintainers of the platform, typically software developers. `hub_admin` is intended for contributors to the CMS and elevated permissions in some REST APIs, for example deleting accounts.

## When to Use

Use when you want to elevate a hub user's permissions.

## Operation

1. Log into Keycloak.
2. Navigate to _eodhp_ realm.
3. Navigate to _Users_ (left side panel), and click on user to elevate.
4. Under _Role mapping_ tab, click _Assign role_.
5. Select _Filter by realm roles_ in drop down.
6. Select the required platform realm role(s), as described in the [realm roles reference](../../reference/iam/realm-roles.md).
7. _Assign_.

The user will now have the new realm role included in their claims, which will allow elevated permissions in certain circumstances.

## Requirements

- Access to Keycloak with permission to assign realm roles

This procedure assigns platform roles only. For Keycloak administration, see [Access the eodhp realm console](access-eodhp-realm-console.md).

## Useful Information

- Users will need to log out and log in again to refresh their OIDC claims with the new role(s).

**Context:** [Authentication and authorization](../../explanation/iam/authentication-and-authorization.md).
