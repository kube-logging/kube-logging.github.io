---
---
> Note: By default, the Logging operator Helm chart doesn't install the `Logging` resource. If you want to install it with Helm, set `logging.enabled` to `true`. The chart can also create `FluentdConfig` and `FluentbitAgent` resources based on `logging.fluentd` and `logging.fluentbit` values respectively.
>
> For details on customizing the installation, see the [Helm chart values](https://github.com/kube-logging/logging-operator/tree/master/charts/logging-operator).
