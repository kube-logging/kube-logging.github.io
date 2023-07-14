---
title: Logging operator with Loki
weight: 200
aliases:
    - /docs/examples/loki-nginx/
---

{{< include-headless "quickstart/intro.md" >}}

{{< include-headless "quickstart-figure-intro.md" >}}

![Configuration architecture](../../img/nginx-loki.png)

{{< include-headless "quickstart/deploy-loki.md" >}}

{{< include-headless "quickstart/deploy-logging-operator-helm.md" >}}

## Configure the Logging operator

1. Create a `Logging` resource to deploy the Fluent Bit log collector agent on the nodes of the cluster, and syslog-ng or Fluentd as the central log forwarder.

    {{< tabpane text=true right=true >}}
    {{% tab header="**Log forwarder**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}
```yaml
  kubectl --namespace logging apply -f - <<"EOF"
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Logging
  metadata:
    name: default-logging-simple
  spec:
    syslogNG: {}
    fluentbit: {}
    controlNamespace: logging
  EOF
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```yaml
  kubectl --namespace logging apply -f - <<"EOF"
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Logging
  metadata:
    name: default-logging-simple
  spec:
    fluentd: {}
    fluentbit: {}
    controlNamespace: logging
  EOF
```
    {{% /tab %}}
    {{< /tabpane >}}

     > Note: You can use `SyslogNGClusterOutput`, `SyslogNGClusterFlow`, `ClusterOutput`, and `ClusterFlow` resources only in the `controlNamespace`. For details, see {{% xref "/docs/configuration/_index.md" %}}.

1. Create an Output resource (called `loki-output` in the example) that configures a Loki output.

    {{< tabpane text=true right=true >}}
    {{% tab header="**Log forwarder**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}
```yaml
kubectl --namespace logging apply -f - <<"EOF"

FIXME

EOF
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```yaml
kubectl --namespace logging apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: loki-output
spec:
  loki:
    url: http://loki:3100
    configure_kubernetes_labels: true
    buffer:
      timekey: 1m
      timekey_wait: 30s
      timekey_use_utc: true
EOF
```
    {{% /tab %}}
    {{< /tabpane >}}

     > Note: In production environment, use a longer `timekey` interval to avoid generating too many objects.

1. Create a `Flow` resource that routes the collected logs to the Loki output.

    <!-- FIXME simplify the example if possible -->

    {{< tabpane text=true right=true >}}
    {{% tab header="**Log forwarder**:" disabled=true /%}}
    {{% tab header="syslog-ng" lang="syslog-ng" %}}
```yaml
kubectl --namespace logging apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: loki-flow
spec:
  FIXME
  match:
    - select:
        labels:
          app.kubernetes.io/name: log-generator
  localOutputRefs:
    - loki-output
EOF
```
    {{% /tab %}}
    {{% tab header="Fluentd" lang="fluentd" %}}
```yaml
kubectl --namespace logging apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: loki-flow
spec:
  filters:
    - tag_normaliser: {}
    - parser:
        remove_key_name_field: true
        reserve_data: true
        parse:
          type: nginx
  match:
    - select:
        labels:
          app.kubernetes.io/name: log-generator
  localOutputRefs:
    - loki-output
EOF
```
    {{% /tab %}}
    {{< /tabpane >}}

1. Install log-generator to produce logs with the label `app.kubernetes.io/name: log-generator`

     ```bash
     helm upgrade --install --wait --create-namespace --namespace logging log-generator kube-logging/log-generator
     ```

1. [Check the collected logs on the Grafana Dashboard](#grafana).

{{< include-headless "quickstart/open-grafana.md" >}}

<!-- FIXME add another simple usecase (filtering, or another namespace), and check the dashboard again -->

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
