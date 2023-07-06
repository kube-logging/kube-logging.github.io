---
---
## Deploy Loki and Grafana

First, deploy Loki and Grafana to your cluster. Loki will store the collected logs, and you can browse the logs using the Grafana dashboard.

1. Add the chart repositories of Loki and Grafana using the following commands:

    ```bash
    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo add loki https://grafana.github.io/loki/charts
    helm repo update
    ```

1. Install Loki into the *logging* namespace:

    ```bash
    helm upgrade --install --create-namespace --namespace logging loki loki/loki
    ```

    > Note: For details on installing Loki, see the [official Grafana Loki Documentation](https://grafana.com/docs/loki/latest/installation/helm/).

1. Install Grafana into the *logging* namespace:

   ```bash
    helm upgrade --install --create-namespace --namespace logging grafana grafana/grafana \
    --set "datasources.datasources\\.yaml.apiVersion=1" \
    --set "datasources.datasources\\.yaml.datasources[0].name=Loki" \
    --set "datasources.datasources\\.yaml.datasources[0].type=loki" \
    --set "datasources.datasources\\.yaml.datasources[0].url=http://loki:3100" \
    --set "datasources.datasources\\.yaml.datasources[0].access=proxy"
    ```
