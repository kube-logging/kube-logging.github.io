---
---
## Deploy the Logging operator with Helm {#helm}

Install the Logging operator and a log-generator application to create sample log messages.

{{< include-headless "deploy-helm-intro.md" >}}

1. Add the chart repository of the Logging operator using the following commands:

    ```bash
    helm repo add kube-logging https://kube-logging.dev/helm-charts
    helm repo update
    ```

1. Install the Logging operator into the *logging* namespace:

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-operator kube-logging/logging-operator
    ```
