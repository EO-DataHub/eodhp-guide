# Adding a Custom Dashboard to Grafana

## Purpose

This guide explains how to add new monitoring dashboards to the EO DataHub Grafana instance. Dashboards are used to visualise metrics from the platform's services, primarily using the Prometheus data source.

## When to Use

Use this guide when you need to create a custom visualisation of system metrics or import an existing dashboard.

## Operation

There are two primary methods for adding a dashboard: directly through the Grafana web interface, or via the platform's configuration in `eodhp-argocd-deployment` repo. [See here](https://github.com/EO-DataHub/eodhp-argocd-deployment/blob/a03ecc8af6a5bb40f88e7537a28ced3f66acce8a/apps/grafana/base/values.yaml#L49)

We recommend developing and testing dashboards in the UI before adding them permanently via helm values configuration.

### Step 1: Ensure You Have Admin Access

All administrative access to Grafana is managed through Keycloak. To create or edit dashboards, your user account must have the `admin` role assigned in Keycloak.

1. Log in to the EO DataHub Keycloak instance.
2. Assign the `admin` role to your user profile.
3. Log out and log back into Grafana to ensure the role takes effect.

### Step 2: Adding a Dashboard via the Grafana UI

1. Navigate to the Grafana instance: https://grafana.eodatahub.org.uk/
2. From the left-hand menu, go to **Dashboards**.
3. In the top right, click **New** to create a new dashboard or **Import** to add one from a JSON file or Grafana.com ID.
    - **New**: Build your dashboard from scratch by adding panels and choosing the Prometheus data source to write queries.
    - **Import**: Paste the dashboard JSON or the Grafana.com dashboard ID. 
4. Save the dashboard.

### Step 3: Adding a Dashboard via Provisioning (Permanent)

This ensures your dashboard is version-controlled and automatically deployed with the platform.

1. **Export the Dashboard JSON:** If you built your dashboard in the UI, you must first export it.

    - Open the dashboard you want to save.
    - elect the 'Export' tab.
    - Click 'Export as JSON' to download the dashboard.

2. **Add the Dashboard to the Grafana Deployment:** The platform's Grafana dashboards are managed as part of its deployment configuration. Add a new entry to the dashboards section of the configuration file, pointing to your dashboard's URL.For example: 

```yaml 
dashboards:  
    ... existing kubernetes dashboards  
    grafana-dashboards-custom:  
        my-new-app-overview:  
        url: <https://raw.githubusercontent.com/your-org/your-repo/main/dashboards/my-new-app.json>  
        token: "" # Add auth token if the repo is private  
```

3. **Deploy the Changes:** Commit your configuration changes to the platform's Git repository. The ArgoCD GitOps pipeline should then automatically redeploy Grafana, which will provision your new dashboard.

## Requirements

- A Keycloak account with the admin role.
- For provisioning: Access to modify and trigger the platform's Grafana deployment configuration.
- For provisioning: A dashboard definition in JSON format accessible via a URL.

## Useful Information

- The primary data source available is **Prometheus**, which scrapes metrics from services from the Kubernetes cluster.
- Dashboards created in the UI are stored in the Grafana database. They may be lost if the Grafana instance is reset or rebuilt. Provisioned dashboards are part of the platform's code and will always be restored.
- The existing `Kubernetes` dashboards already provide excellent examples of how to query the platform's metrics.