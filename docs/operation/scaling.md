---
title: Scaling
weight: 1200
aliases:
    - /docs/one-eye/logging-operator/scaling/
---

In a large-scale infrastructure the logging components can get high load as well. The typical sign of this is when `fluentd` cannot handle its [buffer](../../configuration/plugins/outputs/buffer/) directory size growth for more than the configured or calculated (timekey + timekey_wait) flush interval. In this case, you can [scale the fluentd statefulset]({{< relref "../configuration/fluentd.md#scaling" >}}).

{{< warning >}}
When scaling down Fluentd, the Logging operator does not flush the buffers before terminating the pod. Unless you have a good plan to get the data out from the detached PVC, we don't recommend scaling Fluentd down directly from the Logging operator.

To avoid this problem, you can either:

- write a custom readiness check to get the last pod out from the endpoints of the service, and stop the node only when its buffers are empty, or
- use the commercial [One Eye tool](/docs/one-eye/) that will support this behavior soon.
{{< /warning >}}

> Note: When multiple instances send logs to the same output, the output can receive chunks of messages out of order. Some outputs tolerate this (for example, Elasticsearch), some do not, some require fine tuning (for example, Loki).
