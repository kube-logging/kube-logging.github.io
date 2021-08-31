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

## Enable the default alerting rules {#enable-default-alerts}

Logging operator comes with a number of default alerting rules that help you monitor your logging environment and ensure that it's working properly. To enable the default rules, complete the following steps.

1. Verify that your cluster meets the [Prerequisites](#prerequisites).
1. Enable the alerting rules in your **logging** CR. You can enable alerting separately for Fluentd and Fluent Bit. For example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: default-logging-simple
      namespace: logging
    spec:
      fluentd:
        prometheusRules: true
      fluentbit:
        prometheusRules: true
      controlNamespace: logging
    ```

1. If needed you can [add custom alerting rules](#custom-alerting-rules).

## Overview of default alerting rules

The default alerting rules trigger alerts when:

- Prometheus cannot access the Fluentd node
- Fluentd buffers are quickly filling up
- Traffic to Fluentd is increasing at a high rate
- The number of Fluent Bit or Fluentd errors or retries is high
- Fluentd buffers are over 90% full

Currently, you cannot modify the default alerting rules, because they are generated from the source files. For the detailed list of alerts, see the source code of the [Prometheus alerts for Fluentd](https://github.com/banzaicloud/logging-operator/blob/master/pkg/resources/fluentd/prometheusrules.go) and [Fluent Bit](https://github.com/banzaicloud/logging-operator/blob/master/pkg/resources/fluentbit/prometheusrules.go).

To enable these alerts on your cluster, see [Enable the default alerting rules](#enable-default-alerts).

## Add custom alerting rules {#custom-alerting-rules}

Although you cannot modify the default alerting rules, you can add your own custom rules to the cluster by creating and applying [AlertmanagerConfig resources to the Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/alerting.md).
