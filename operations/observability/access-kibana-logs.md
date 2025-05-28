# Access Kibana Logs

## Purpose

The ELK stack has been installed to the platform to provide observability of platform logs and metrics. Kibana (part of the ELK stack) can be used to visualise the logs of the platform and gain an insight into the behaviour. This operation will detail how to gain access to the Kibana UI. It will not go into the details of how to configure dashboards to view logs, in-depth information on how to do this is readily available online.

## When to Use

Use this procedure when you need to gain access to the Kibana UI, deployed as part of the platform.

## Operation

1. For initial access to Kibana, retrieve the default user credentials from the Kubernetes cluster. The default username is `elastic`, and the password can be obtained using command below.
   ```sh
   kubectl -n keycloak get secret keycloak-initial-admin -o jsonpath="{.data.elastic}" | base64 -d; echo
   ```
2. Navigate to https://logs.eodatahub.org.uk and log in with credentials.
3. From here, you can use the Analytics and Observability apps to view logs and metrics for the platform.
4. You can configure dashboards displaying the information you need so you can understand the behaviour of the platform at a glance.

## Requirements

- Access to the Kubernetes cluster via kubectl

## Useful Information

- The ECK operator requires an enterprise licence for its OIDC integration functionality. It is therefore necessary to use the pre-configured authentication system, separate from the Hub platform's OIDC provider.
- Configuring useful dashboards to display metrics and specific logs by index is a complex topic and highly use-case specific. Describing how to do this is left to the online documentation of Kibana.
