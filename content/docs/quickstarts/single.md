---
title: Single app, one destination
weight: 100
---

{{< include-headless "quickstart/intro.md" >}}

In this tutorial, you will:

- Install the Logging operator on a cluster.
- Configure Logging operator to collect logs from a namespace and send it to an sample output.
- Install a sample application (log-generator) to collect its logs.
- Check the collected logs.

{{< include-headless "quickstart/deploy-logging-operator-helm.md" >}}

## Configure the Logging operator

1. Create a `Logging` resource to deploy syslog-ng or Fluentd as the central log aggregator and forwarder. You can complete this quick start guide with any of them, but they have different features, so they are not equivalent. For details, see {{% xref "/docs/configuration/fluentd-vs-syslog-ng" %}}.

    Run one of the following commands.

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

    > Note: The control namespace is where the Logging operator deploys the forwarder's resources, like the StatefulSet and the configuration secrets. Usually it's called `logging`.
    >
    > By default, this namespace is used to define the cluster-wide resources: `SyslogNGClusterOutput`, `SyslogNGClusterFlow`, `ClusterOutput`, and `ClusterFlow`. For details, see {{% xref "/docs/configuration/_index.md" %}}.

    Expected output:

    ```shell
    logging.logging.banzaicloud.io/quickstart created
    ```

1. Create a `FluentbitAgent` resource to collect logs from all containers. No special configuration is required for now.

    ```bash
    kubectl --namespace logging apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: FluentbitAgent
    metadata:
        name: quickstart
    spec: {}
    EOF
    ```

    Expected output:

    ```shell
    fluentbitagent.logging.banzaicloud.io/quickstart created
    ```

1. Check that the resources were created successfully so far. Run the following command:

    ```shell
    kubectl get pod --namespace logging --selector app.kubernetes.io/managed-by=quickstart
    ```

    You should already see a completed configcheck pod that validates the forwarder's configuration before the actual statefulset starts.
    There should also be a running fluentbit instance per node, that already starts to send all logs to the forwarder.
    
    {{< tabpane text=true right=true >}}
    {{% tab header="**All logging resources**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}
```shell
NAME                                        READY   STATUS      RESTARTS   AGE
quickstart-fluentbit-jvdp5                  1/1     Running     0          3m5s
quickstart-syslog-ng-0                      2/2     Running     0          3m5s
quickstart-syslog-ng-configcheck-8197c552   0/1     Completed   0          3m42s
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```shell
NAME                                      READY   STATUS      RESTARTS   AGE
quickstart-fluentbit-nk9ms                1/1     Running     0          19s
quickstart-fluentd-0                      2/2     Running     0          19s
quickstart-fluentd-configcheck-ac2d4553   0/1     Completed   0          60s
```
    {{% /tab %}}
    {{< /tabpane >}}

1. Create a namespace (for example, `quickstart`) from where you want to collect the logs.

    ```bash
    kubectl create namespace quickstart
    ```

    Expected output:

    ```shell
    namespace/quickstart created
    ```

1. Create a [flow]({{% relref "/docs/configuration/flow.md" %}}) and an [output]({{< relref "/docs/configuration/output.md" >}}) resource in the same namespace (`quickstart`). The flow resource routes logs from the namespace to a specific output. In this example, the output is called `http`. The flow resources are called `SyslogNGFlow` and `Flow`, the output resources are `SyslogNGOutput` and `Output` for syslog-ng and Fluentd, respectively.

    {{< tabpane text=true right=true >}}
    {{% tab header="**Flow and output**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}

```yaml
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
    disk_buffer:
      dir: /buffers
      disk_buf_size: 512000000 # 512 MB
      reliable: true
EOF
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```yaml
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

    > Note: In production environment, use a longer `timekey` interval to avoid generating too many objects.

    Expected output:

    {{< tabpane text=true right=true >}}
    {{% tab header="**Log forwarder**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}
```shell
syslogngflow.logging.banzaicloud.io/log-generator created
syslogngoutput.logging.banzaicloud.io/http created
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```shell
flow.logging.banzaicloud.io/log-generator created
output.logging.banzaicloud.io/http created
```
    {{% /tab %}}
    {{< /tabpane >}}

1. Check that the resources were created successfully. Run the following command:

    ```shell
    kubectl get logging-all --namespace quickstart
    ```

    You should see that the logging resource has been created and the flow and output are active.

    {{< tabpane text=true right=true >}}
    {{% tab header="**All logging resources**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}

```shell
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

1. Install log-generator to produce logs with the label `app.kubernetes.io/name: log-generator`

     ```bash
     helm upgrade --install --wait --namespace quickstart log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator
     ```

    Expected output:

    ```shell
    Release "log-generator" does not exist. Installing it now.
    Pulled: ghcr.io/kube-logging/helm-charts/log-generator:0.7.0
    Digest: sha256:0eba2c5c3adfc33deeec1d1612839cd1a0aa86f30022672ee022beab22436e04
    NAME: log-generator
    LAST DEPLOYED: Tue Aug 15 16:21:40 2023
    NAMESPACE: quickstart
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    ```

    The log-generator application starts to create HTTP access logs. Logging operator collects these log messages and sends them to the test-receiver pod defined in the output custom resource.

1. Check that the logs are delivered to the test-receiver pod output. First, run the following command to get the name of the test-receiver pod:

    ```shell
    kubectl logs --namespace logging -f svc/logging-operator-test-receiver
    ```

    The output should be similar to the following:

    {{< tabpane text=true right=true >}}
    {{% tab header="**Receiving logs**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}

```bash
[0] http.0: [[1692117678.581721054, {}], {"ts"=>"2023-08-15T16:41:18.130862Z", "time"=>"2023-08-15T16:41:18.13086297Z", "stream"=>"stdout", "log"=>"142.251.196.69 - - [15/Aug/2023:16:41:18 +0000] "PUT /index.html HTTP/1.1" 302 24666 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36" "-"", "kubernetes"=>{"pod_name"=>"log-generator-56b7dfb79-6v67b", "pod_id"=>"b7e8a5b2-9164-46d1-ba0a-8d142bdfb4cb", "namespace_name"=>"quickstart", "labels"=>{"pod-template-hash"=>"56b7dfb79", "app.kubernetes.io/name"=>"log-generator", "app.kubernetes.io/instance"=>"log-generator"}, "host"=>"minikube", "docker_id"=>"fe60b1c0fdf97f062ed91e3a2074caf3ee3cb4f3d12844f2c6f5d8212419907d", "container_name"=>"log-generator", "container_image"=>"ghcr.io/kube-logging/log-generator:0.7.0", "container_hash"=>"ghcr.io/kube-logging/log-generator@sha256:e26102ef2d28201240fa6825e39efdf90dec0da9fa6b5aea6cf9113c0d3e93aa"}}]
```

    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```bash
[0] http.0: [[1692118483.267342676, {}], {"log"=>"51.196.131.145 - - [15/Aug/2023:16:54:36 +0000] "PUT / HTTP/1.1" 200 7823 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36" "-"", "stream"=>"stdout", "time"=>"2023-08-15T16:54:36.019636047Z", "kubernetes"=>{"pod_name"=>"log-generator-56b7dfb79-rrzsz", "namespace_name"=>"quickstart", "pod_id"=>"902dc881-af36-4054-b377-47e2d751e6cd", "labels"=>{"app.kubernetes.io/instance"=>"log-generator", "app.kubernetes.io/name"=>"log-generator", "pod-template-hash"=>"56b7dfb79"}, "host"=>"minikube", "container_name"=>"log-generator", "docker_id"=>"7615c4c72d8fdd05137dc9845204d7ef681b750b6f2a6d27bd75190b12dc5d8e", "container_hash"=>"ghcr.io/kube-logging/log-generator@sha256:e26102ef2d28201240fa6825e39efdf90dec0da9fa6b5aea6cf9113c0d3e93aa", "container_image"=>"ghcr.io/kube-logging/log-generator:0.7.0"}}]
```
    {{% /tab %}}
    {{< /tabpane >}}

    The log messages include the usual information of the access logs, and also Kubernetes-specific information like the pod name, labels, and so on.

## Summary

If you have completed this guide, you have made the following changes to your cluster:

- Installed the Fluent Bit agent on every node of the cluster to collect the logs and the labels from the node.
- Installed syslog-ng or Fluentd on the cluster, to receive the logs from the Fluent Bit agents, and filter, parse, and transform them as needed, and to route the incoming logs to an output. To learn more about routing and filtering, see {{% xref "/docs/configuration/log-routing-syslog-ng.md" %}} or {{% xref "/docs/configuration/log-routing.md" %}}. - Created the following resources that configure Logging operator and the components it manages:

    - `Logging` to configure the logging infrastructure, like the details of the Fluent Bit and the syslog-ng or Fluentd deployment. To learn more about configuring the logging infrastructure, see {{% xref "/docs/logging-infrastructure/_index.md" %}}.
    - `SyslogNGOutput` or `Output` to define an http output that receives the collected messages. To learn more, see {{% xref "/docs/configuration/output.md#syslogngoutput" %}} or {{% xref "/docs/configuration/output.md" %}}.
    - `SyslogNGFlow` or `Flow` that processes the incoming messages and routes them to the appropriate output. To learn more, see {{% xref "/docs/configuration/flow.md#syslogngflow" %}} or {{% xref "/docs/configuration/flow.md" %}}.

- Installed a simple receiver to act as the destination of the logs, and configured the the log forwarder to send the logs from the `quickstart` namespace to this destination.
- Installed a log-generator application to generate sample log messages, and verified that the logs of this application arrive to the output.

<!-- FIXME next steps/links to examples -->

{{< include-headless "support-troubleshooting.md" >}}
