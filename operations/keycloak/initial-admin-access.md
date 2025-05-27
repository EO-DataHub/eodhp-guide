# Keycloak Initial Admin Access

## Purpose

When Keycloak is first instantiated, platform operators will need to gain access to the admin panel to configure the realm. This operation describes how to obtain initial admin access.

The initial admin user should be replaced with an admin user created by a hub admin.

## When to Use

Execute this procedure when a fresh installation of Keycloak has occurred.

## Operation

1. Obtain the initial admin credentials from the Kubernetes cluster.

   ```sh
   kubectl -n keycloak get secret keycloak-initial-admin -o jsonpath="{.data.username}" | base64 -d; echo
   kubectl -n keycloak get secret keycloak-initial-admin -o jsonpath="{.data.password}" | base64 -d; echo
   ```

2. Visit https://eodatahub.org.uk/keycloak and log in. You will be logged into the master realm.
3. Create a new user in the master realm (Users > Add User). You must provide a Username, Email, First and Last Names, and also check "Email verified". Create the user.
4. Click on the user under Users and use the Credentials tab to assign a password. Keep this password secure.
5. Under Role mapping tab Assign role, Filter by realm roles and Assign "admin" role.
6. Log out of initial admin user and log in as your new user. Confirm you have admin access by, for instance, creating a new realm (delete it afterwards).
7. Once you have confirmed you have admin access using your new user, delete the initial admin user created by Keycloak.

## Requirements

- Access to Kubernetes cluster using `kubectl`

## Useful Information

- The default Keycloak user should be deleted after initial access has been achieved. This procedure will only allow access while the default user still exists.
