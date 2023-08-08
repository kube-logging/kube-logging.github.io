
## Deploy the Logging operator with Helm {#helm}

{{< include-headless "deploy-helm-intro.md" >}}

```
helm upgrade --install --wait \
     --create-namespace --namespace logging \
     --set testReceiver.enabled=true \
     logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
```

This command will install the latest stable Logging Operator and an extra workload (service and deployment) 
with the name `logging-operator-test-receiver` which listens on an HTTP port and receives JSON messages,
which it writes to standard out so that it is trivial to observe.

The following pods and services are expected to run after running the above command:
```shell
kubectl get deploy -n logging
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
logging-operator                 1/1     1            1           15m
logging-operator-test-receiver   1/1     1            1           15m

kubectl get svc -n logging
NAME                             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
logging-operator                 ClusterIP   None           <none>        8080/TCP   15m
logging-operator-test-receiver   ClusterIP   10.99.77.113   <none>        8080/TCP   15m
```
