---
title: Single app, one destination
weight: 100
---

{{< include-headless "quickstart/intro.md" >}}

{{< include-headless "quickstart/deploy-logging-operator-helm.md" >}}

## Configure the Logging operator


1. Create a `Logging` resource to deploy syslog-ng or Fluentd as the central log aggregator and forwarder. For a hint on syslog-ng vs fluentd, see {{% xref "/docs/configuration/fluentd-vs-syslog-ng" %}} 

    {{< tabpane text=true right=true >}}
    {{% tab header="**Log forwarder**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}
```yaml
kubectl --namespace logging apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: quickstart
spec:
  controlNamespace: logging
  syslogNG:
    # `#` is the recommended key delimiter when parsing json in syslog-ng
    jsonKeyDelim: '#'
EOF
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```yaml
kubectl --namespace logging apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: quickstart
spec:
  controlNamespace: logging
  fluentd:
    disablePvc: true
EOF
```
    {{% /tab %}}
    {{< /tabpane >}}

     > The control namespace is where the logging operator will deploy the forwarder's resources, like the statefulset and the configuration secrets.
       By default this namespace is used to define cluster-wide resources: `SyslogNGClusterOutput`, `SyslogNGClusterFlow`, `ClusterOutput`, `ClusterFlow`
       For details, see {{% xref "/docs/configuration/_index.md" %}}.

1. Create `FluentbitAgent` to collect logs from all containers. No special configuration is required for now.

    ```bash
    kubectl --namespace logging apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: FluentbitAgent
    metadata:
        name: quickstart
    spec: {}
    EOF
    ```

1. What you should see at this point?

    You should already see a completed configcheck pod, that validates the forwarder's configuration before the actual statefulset starts.
    There should also be a fluentbit instance per node running, that already starts to send all logs to the forwarder.
    
   {{< tabpane text=true right=true >}}
   {{% tab header="**All logging resources**:" disabled=true /%}}
   {{% tab header="syslog-ng" lang="syslog-ng" %}}
```shell
kubectl get pod -l app.kubernetes.io/managed-by=quickstart
NAME                                        READY   STATUS      RESTARTS   AGE
quickstart-fluentbit-jvdp5                  1/1     Running     0          3m5s
quickstart-syslog-ng-0                      2/2     Running     0          3m5s
quickstart-syslog-ng-configcheck-8197c552   0/1     Completed   0          3m42s
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```shell
kubectl get pod -n logging -l app.kubernetes.io/managed-by=quickstart

NAME                                      READY   STATUS      RESTARTS   AGE
quickstart-fluentbit-nk9ms                1/1     Running     0          19s
quickstart-fluentd-0                      2/2     Running     0          19s
quickstart-fluentd-configcheck-ac2d4553   0/1     Completed   0          60s
```
    {{% /tab %}}
    {{< /tabpane >}}

1. Create a [flow]({{% relref "/docs/configuration/flow.md" %}}) resource that actually routes the logs of a specific application - from the same namespace the flow is in - to a specific [output]({{< relref "/docs/configuration/output.md" >}}), which we will call `http`.

   {{< tabpane text=true right=true >}}
   {{% tab header="**Flow and output**:" disabled=true /%}}
   {{% tab header="syslog-ng" lang="syslog-ng" %}}
```yaml
kubectl create namespace quickstart --dry-run=client -o yaml | kubectl apply -f-
kubectl --namespace quickstart apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: log-generator
spec:
  match:
    regexp:
      value: "json#kubernetes#labels#app.kubernetes.io/instance"
      pattern: log-generator
      type: string
  localOutputRefs:
    - http
---
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGOutput
metadata:
  name: http
spec:
  http:
    url: http://logging-operator-test-receiver:8080
    headers:
      - "Content-Type: application/json"
    body: "$(format-json --subkeys json# --key-delimiter #)"
EOF
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```yaml
kubectl create namespace quickstart --dry-run=client -o yaml | kubectl apply -f-
kubectl --namespace quickstart apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: log-generator
spec:
  match:
    - select:
        labels:
          app.kubernetes.io/name: log-generator
  localOutputRefs:
    - http
---
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: http
spec:
  http:
    endpoint: http://logging-operator-test-receiver:8080
    content_type: application/json
    buffer:
      type: memory
      tags: time
      timekey: 1s
      timekey_wait: 0s
EOF
```
    {{% /tab %}}
    {{< /tabpane >}}

1. What you should see at this point?

    You should see that logging resource have been created and the flow and output are active.

   {{< tabpane text=true right=true >}}
   {{% tab header="**All logging resources**:" disabled=true /%}}
   {{% tab header="syslog-ng" lang="syslog-ng" %}}
```shell
kubectl get logging-all -n quickstart
NAME                                               AGE
fluentbitagent.logging.banzaicloud.io/quickstart   10m

NAME                                        AGE
logging.logging.banzaicloud.io/quickstart   10m

NAME                                         ACTIVE   PROBLEMS
syslogngoutput.logging.banzaicloud.io/http   true

NAME                                                ACTIVE   PROBLEMS
syslogngflow.logging.banzaicloud.io/log-generator   true
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```shell
kubectl get logging-all -n quickstart

NAME                                        ACTIVE   PROBLEMS
flow.logging.banzaicloud.io/log-generator   true

NAME                                 ACTIVE   PROBLEMS
output.logging.banzaicloud.io/http   true

NAME                                        AGE
logging.logging.banzaicloud.io/quickstart   3m12s

NAME                                               AGE
fluentbitagent.logging.banzaicloud.io/quickstart   3m2s
```
    {{% /tab %}}
    {{< /tabpane >}}

1. Finally install log-generator to produce logs with the label `app.kubernetes.io/name: log-generator`

     ```bash
     helm upgrade --install --wait --namespace quickstart log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator
     ```

1. You should see logs flowing to our test receiver

   {{< tabpane text=true right=true >}}
   {{% tab header="**Receiving logs**:" disabled=true /%}}
   {{% tab header="syslog-ng" lang="syslog-ng" %}}
```bash
kubectl logs -n logging -f svc/logging-operator-test-receiver
[0] http.0: [[1692117678.581721054, {}], {"ts"=>"2023-08-15T16:41:18.130862Z", "time"=>"2023-08-15T16:41:18.13086297Z", "stream"=>"stdout", "log"=>"142.251.196.69 - - [15/Aug/2023:16:41:18 +0000] "PUT /index.html HTTP/1.1" 302 24666 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36" "-"", "kubernetes"=>{"pod_name"=>"log-generator-56b7dfb79-6v67b", "pod_id"=>"b7e8a5b2-9164-46d1-ba0a-8d142bdfb4cb", "namespace_name"=>"quickstart", "labels"=>{"pod-template-hash"=>"56b7dfb79", "app.kubernetes.io/name"=>"log-generator", "app.kubernetes.io/instance"=>"log-generator"}, "host"=>"minikube", "docker_id"=>"fe60b1c0fdf97f062ed91e3a2074caf3ee3cb4f3d12844f2c6f5d8212419907d", "container_name"=>"log-generator", "container_image"=>"ghcr.io/kube-logging/log-generator:0.7.0", "container_hash"=>"ghcr.io/kube-logging/log-generator@sha256:e26102ef2d28201240fa6825e39efdf90dec0da9fa6b5aea6cf9113c0d3e93aa"}}]
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```bash
kubectl logs -n logging -f svc/logging-operator-test-receiver
[0] http.0: [[1692118483.267342676, {}], {"log"=>"51.196.131.145 - - [15/Aug/2023:16:54:36 +0000] "PUT / HTTP/1.1" 200 7823 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36" "-"", "stream"=>"stdout", "time"=>"2023-08-15T16:54:36.019636047Z", "kubernetes"=>{"pod_name"=>"log-generator-56b7dfb79-rrzsz", "namespace_name"=>"quickstart", "pod_id"=>"902dc881-af36-4054-b377-47e2d751e6cd", "labels"=>{"app.kubernetes.io/instance"=>"log-generator", "app.kubernetes.io/name"=>"log-generator", "pod-template-hash"=>"56b7dfb79"}, "host"=>"minikube", "container_name"=>"log-generator", "docker_id"=>"7615c4c72d8fdd05137dc9845204d7ef681b750b6f2a6d27bd75190b12dc5d8e", "container_hash"=>"ghcr.io/kube-logging/log-generator@sha256:e26102ef2d28201240fa6825e39efdf90dec0da9fa6b5aea6cf9113c0d3e93aa", "container_image"=>"ghcr.io/kube-logging/log-generator:0.7.0"}}]
```
    {{% /tab %}}
    {{< /tabpane >}}

## Summary

If you have completed this guide, you have made the following changes to your cluster:

- Installed the Fluent Bit agent on every node of the cluster that collects the logs and the labels from the node.
- Installed syslog-ng or Fluentd on the cluster, to receive the logs from the Fluent Bit agents, and filter, parse, and transform them as needed, and to route the incoming logs to an output. To learn more about routing and filtering, see {{% xref "/docs/configuration/log-routing-syslog-ng.md" %}} or {{% xref "/docs/configuration/log-routing.md" %}}.
- Installed [Grafana Loki](https://grafana.com/docs/loki/latest/) to store your logs, and configured Logging operator to send the incoming logs to the Loki output.
- Installed [Grafana](https://grafana.com/docs/grafana/latest/) that allows you to browse the logs stored in Loki.
- Created the following resources that configure Logging operator and the components it manages:

    - `Logging` to configure the logging infrastructure, like the details of the Fluent Bit and the syslog-ng or Fluentd deployment. To learn more about configuring the logging infrastructure, see {{% xref "/docs/logging-infrastructure/_index.md" %}}.
    - `SyslogNGOutput` or `Output` to define a Loki output that receives the collected messages. To learn more, see {{% xref "/docs/configuration/output.md#syslogngoutput" %}} or {{% xref "/docs/configuration/output.md" %}}.
    - `SyslogNGFlow` or `Flow` that processes the incoming messages and routes them to the appropriate output. To learn more, see {{% xref "/docs/configuration/flow.md#syslogngflow" %}} or {{% xref "/docs/configuration/flow.md" %}}.

{{< include-headless "support-troubleshooting.md" >}}
