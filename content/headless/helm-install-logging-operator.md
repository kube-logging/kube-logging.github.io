Install the Logging operator into the *logging* namespace:

```bash
helm upgrade --install --wait --create-namespace --namespace logging logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
```
