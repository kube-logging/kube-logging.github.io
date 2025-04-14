---
title: Scaling
weight: 1200
aliases:
    - /docs/one-eye/logging-operator/scaling/
---

> Note: When multiple instances send logs to the same output, the output can receive chunks of messages out of order. Some outputs tolerate this (for example, Elasticsearch), some do not, some require fine tuning (for example, Loki).

## Scaling Fluentd

In a large-scale infrastructure the logging components can get high load as well. The typical sign of this is when `fluentd` cannot handle its [buffer](../../configuration/plugins/outputs/buffer/) directory size growth for more than the configured or calculated (timekey + timekey_wait) flush interval. In this case, you can [scale the fluentd statefulset]({{< relref "../logging-infrastructure/fluentd.md#scaling" >}}).

The Logging operator supports scaling a **Fluentd aggregator** statefulset up and down. Scaling statefulset pods down is challenging, because we need to take care of the underlying volumes with buffered data that hasn't been sent, but the Logging operator supports that use case as well.

The details for that and how to configure an HPA is described in the following documents:
- https://github.com/kube-logging/logging-operator/blob/master/docs/volume-drainer.md
- https://github.com/kube-logging/logging-operator/blob/master/docs/scaling.md

## Scaling SyslogNG

SyslogNG can be scaled up as well, but persistent disk buffers are not processed automatically when scaling the statefulset down. That is currently a manual process.
