# Elevate User

## Purpose

This operation is required when you want to elevate a user's permission in the platform. There are two tiers; `admin` and `hub_admin`.

`admin` is intended for maintainers of the platform, typically softwre developers.

`hub_admin` is intended for contributers to the CMS and elevated permissions in some REST APIs, e.g. deleting accounts.

## When to Use

Use when you want to elevate a hub user's permissions.

## Operation

1. Log into Keycloak.
2. Navigate to _eodhp_ realm.
3. Navigate to _Users_ (left side panel), and click on user to elevate.
4. Under _Role mapping_ tab, click _Assign role_.
5. Select _Filter by realm roles_ in drop down.
6. Select either `admin` and/or `hub_admin`, as required.
7. _Assign_.

The user will now have the new realm role included in their claims, which will allow elevated permissions in certain circumstances.

## Requirements

- Admin access to Keycloak

## Useful Information

- Users will need to log out and log in again to refresh their OIDC claims with the new role(s).
