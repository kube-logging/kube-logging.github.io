---
title: Alerting
weight: 900
---

This section describes how to set alerts for your logging infrastructure. Alternatively, you can enable the default alerting rules that are provided by the Logging operator.

> Note: Alerting based on the contents of the collected log messages is not covered here.

## Prerequisites

Using alerting rules requires the following:

- Logging operator 3.14.0 or newer installed on the cluster.
- Prometheus operator installed on the cluster. For details, see {{% xref "logging-operator-monitoring.md" %}}.
- {{< include-headless "syslog-ng-minimum-version.md" >}}

## Enable the default alerting rules {#enable-default-alerts}

Logging operator comes with a number of default alerting rules that help you monitor your logging environment and ensure that it's working properly. To enable the default rules, complete the following steps.

1. Verify that your cluster meets the [Prerequisites](#prerequisites).
1. Enable the alerting rules in your **logging** CR. You can enable alerting separately for Fluentd, syslog-ng, and Fluent Bit. For example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: default-logging-simple
      namespace: logging
    spec:
      fluentd:
        metrics:
          prometheusRules: true
      fluentbit:
        metrics:
          prometheusRules: true
      syslogNG:
        metrics:
          prometheusRules: true
      controlNamespace: logging
    ```

1. If needed you can [add custom alerting rules](#custom-alerting-rules).

## Overview of default alerting rules

The default alerting rules trigger alerts when:

For the **Fluent Bit** log collector:

- The number of Fluent Bit errors or retries is high

For the **Fluentd** and **syslog-ng** log forwarders:

- Prometheus cannot access the log forwarder node
- The buffers of the log forwarder are filling up quickly
- Traffic to the log forwarder is increasing at a high rate
- The number of errors or retries is high on the log forwarder
- The buffers are over 90% full

For the detailed list of default alerts, see the source code:

- For [Fluentd](https://github.com/kube-logging/logging-operator/blob/master/pkg/resources/fluentd/prometheusrules.go)
- For [syslog-ng](https://github.com/kube-logging/logging-operator/blob/master/pkg/resources/syslogng/prometheusrules.go)
- For [Fluent Bit](https://github.com/kube-logging/logging-operator/blob/master/pkg/resources/fluentbit/prometheusrules.go)

To enable these alerts on your cluster, see [Enable the default alerting rules](#enable-default-alerts).

## Customize alerting rules {#customize-alerting-rules}

Use the `prometheusRulesOverride` option to modify built-in alerting rules (for example, change the severity or threshold) or add new custom rules.

The `prometheusRulesOverride` option is available for both regular metrics and buffer volume metrics:

- `metrics.prometheusRulesOverride`: Alerting rules for log forwarder metrics.
- `bufferVolumeMetrics.prometheusRulesOverride`: Alerting rules for buffer volume metrics.

For example, to change the severity of a buffer volume alert from `critical` to `warning`:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
  namespace: logging
spec:
  fluentd:
    bufferVolumeMetrics:
      prometheusRules: true
      prometheusRulesOverride:
      - alert: FluentdBufferSize
        labels:
          severity: warning
  controlNamespace: logging
```

You can also add new alerting rules. For example, to add a custom alert for predicted buffer growth:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
  namespace: logging
spec:
  syslogNG:
    bufferVolumeMetrics:
      prometheusRules: true
      prometheusRulesOverride:
      - alert: SyslogNGPredictedBufferGrowth
        expr: predict_linear(node_filesystem_avail_bytes{mountpoint="/buffers"}[6h], 4*3600) < 0
        for: 1h
        labels:
          service: syslog-ng
          severity: warning
        annotations:
          summary: Syslog-NG buffer predicted to fill up in 4 hours.
          description: Based on recent buffer growth, the buffer volume is predicted to fill up within 4 hours.
  controlNamespace: logging
```

For the list of fields you can set in `prometheusRulesOverride`, see [PrometheusRulesOverride]({{< relref "/docs/configuration/crds/v1beta1/common_types.md#prometheusrulesoverride" >}}).

## Add custom alerting rules {#custom-alerting-rules}

You can also add your own custom rules to the cluster by creating and applying [AlertmanagerConfig resources to the Prometheus Operator](hhttps://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/developer/alerting.md).

For example, the Logging operator creates the following alerting rule to detect if a Fluentd node is down:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
  name: logging-demo-fluentd-metrics
  namespace: logging
spec:
  groups:
  - name: fluentd
    rules:
    - alert: FluentdNodeDown
      annotations:
        description: Prometheus could not scrape {{ "{{ $labels.job }}" }} for more
          than 30 minutes
        summary: fluentd cannot be scraped
      expr: up{job="logging-demo-fluentd-metrics", namespace="logging"} == 0
      for: 10m
      labels:
        service: fluentd
        severity: critical
```

On the Prometheus web interface, this rule looks like:

![Fluentd alerting rule on the Prometheus web interface](alerting-rule-in-prometheus.png)
