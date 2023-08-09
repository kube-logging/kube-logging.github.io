Install the Logging operator into the *logging* namespace:

```bash
helm upgrade --install --wait --create-namespace --namespace logging logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
```

Expected output:

```shell
Release "logging-operator" does not exist. Installing it now.
Pulled: ghcr.io/kube-logging/helm-charts/logging-operator:4.3.0
Digest: sha256:c2ece861f66a3a2cb9788e7ca39a267898bb5629dc98429daa8f88d7acf76840
NAME: logging-operator
LAST DEPLOYED: Wed Aug  9 11:02:12 2023
NAMESPACE: logging
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
