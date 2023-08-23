
## Deploy the Logging operator with Helm {#helm}

{{< include-headless "deploy-helm-intro.md" >}}

This command installs the latest stable Logging operator and an extra workload (service and deployment). This workload is called `logging-operator-test-receiver`. It listens on an HTTP port, receives JSON messages, and writes them to the standard output (stdout) so that it is trivial to observe.

```shell
helm upgrade --install --wait \
     --create-namespace --namespace logging \
     --set testReceiver.enabled=true \
     logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
```

Expected output:

```shell
Release "logging-operator" does not exist. Installing it now.
Pulled: ghcr.io/kube-logging/helm-charts/logging-operator:4.3.0
Digest: sha256:c2ece861f66a3a2cb9788e7ca39a267898bb5629dc98429daa8f88d7acf76840
NAME: logging-operator
LAST DEPLOYED: Tue Aug 15 15:58:41 2023
NAMESPACE: logging
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

After the installation, check that the following pods and services are running:

```shell
kubectl get deploy -n logging
```

Expected output:

```shell
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
logging-operator                 1/1     1            1           15m
logging-operator-test-receiver   1/1     1            1           15m
```

```shell
kubectl get svc -n logging
```

Expected output:

```shell
NAME                             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
logging-operator                 ClusterIP   None           <none>        8080/TCP   15m
logging-operator-test-receiver   ClusterIP   10.99.77.113   <none>        8080/TCP   15m
```
