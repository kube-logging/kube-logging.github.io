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
- {{< include-headless "syslog-ng-minimum-version.md" "one-eye/logging-operator" >}}

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

Currently, you cannot modify the default alerting rules, because they are generated from the source files. For the detailed list of alerts, see the source code:

- For [Fluentd](https://github.com/banzaicloud/logging-operator/blob/master/pkg/resources/fluentd/prometheusrules.go)
- For [syslog-ng](https://github.com/banzaicloud/logging-operator/blob/master/pkg/resources/syslogng/prometheusrules.go)
- For [Fluent Bit](https://github.com/banzaicloud/logging-operator/blob/master/pkg/resources/fluentbit/prometheusrules.go)

To enable these alerts on your cluster, see [Enable the default alerting rules](#enable-default-alerts).

## Add custom alerting rules {#custom-alerting-rules}

Although you cannot modify the default alerting rules, you can add your own custom rules to the cluster by creating and applying [AlertmanagerConfig resources to the Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/alerting.md).

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

![Fluentd alerting rule on the Prometheus web interface](../alerting-rule-in-prometheus.png)
