---
title: Access the eodhp realm console
doc_status: needs-verification
tags:
  - keycloak
  - identity
last_reviewed:
reviewed_by:
review_notes:
---
# Access the eodhp realm console

Use this procedure for routine administration of the `eodhp` realm with a personally attributable account. It does not cover initial Keycloak setup.

## Requirements

- A supported personal Hub account or identity-provider account.
- An administrator has already assigned the required `realm-management` client roles to that account.
- Access to the relevant platform environment.

## Access the console

1. Visit `https://eodatahub.org.uk/keycloak/admin/eodhp/console/` (use the equivalent host for test or staging).
2. Sign in with your personal account and supported identity provider.
3. Confirm that the selected realm is **eodhp** before making changes.
4. Perform the required operation. Do not use a shared account.

## Grant console access

An already-authorized Keycloak administrator should:

1. Open the `eodhp` realm and go to **Users**.
2. Select the person's account and open **Role mapping**.
3. Under **Client roles**, select `realm-management`.
4. Assign only the client roles needed for the task.

Use `query-users` or `view-users` for user lookup where those permissions are sufficient. `manage-users` may be required for user or role-mapping administration. The exact operation permitted by each role depends on the deployed Keycloak version and configuration; verify the required permission before assigning it. `realm-admin` provides broad realm administration and should not be the default for a narrow task.

Platform realm roles such as `admin`, `hub_admin`, and `grafana-viewer` are not substitutes for these client roles.

## Verify and revoke access

After a role is assigned, sign out and sign in again so that the session and token contain the new permissions. Verify access with a non-destructive operation, such as viewing the realm or locating an existing user. Do not create test users or realms solely to test access.

To revoke access, remove the delegated `realm-management` client roles from the user's **Role mapping** page. Ask the user to sign out; active sessions or tokens may remain valid until they expire or are revoked according to the platform's session settings.

## Troubleshooting

- Check that the URL points to the intended environment and that **eodhp** is selected, rather than `master`.
- If the console is visible but an operation is denied, the account is missing the relevant `realm-management` client role.
- If a newly assigned role has no effect, sign out and sign in again.
- If the required permission is unclear, ask a Keycloak administrator to confirm it for the deployed version before granting broader access.

**Related:** [EODH realm roles](../../reference/iam/realm-roles.md) · [Keycloak initial admin access](keycloak-initial-admin-access.md) · [Keycloak](../../reference/services/keycloak.md)
