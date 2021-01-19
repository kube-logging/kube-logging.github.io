---
title: Logging custom resource
shorttitle: Logging
weight: 20
---

The `logging` resource defines the logging infrastructure for your cluster that collects and transports your log messages, and also contains configurations for Fluentd and Fluent-bit. It also establishes the `controlNamespace`, the administrative namespace of the Logging operator. The Fluentd statefulset and Fluent-bit daemonset are deployed in this namespace, and global resources like `ClusterOutput` and `ClusterFlow` are evaluated only in this namespace by default - they are ignored in any other namespace unless `allowClusterResourcesFromAllNamespaces` is set to true.

You can define multiple `logging` resources if needed, for example, if you want to run multiple fluentd instances with separate configurations.
You can customize the fluentd and fluent-bit configuration in the logging resource). It also declares `watchNamespaces` if applicable to narrow down the namespaces in which the logging operator should evaluate and incorporate`Flow` and `Output` resources into fluentd's configuration.

You can install a `logging` resource with built-in TLS generation using the [logging Helm chart](https://github.com/banzaicloud/logging-operator/tree/master/charts/logging-operator-logging).

> The [One Eye](/products/one-eye/) observability tool can [manage the TLS certificates of the logging resource](/docs/one-eye/tls/) using cert-manager.

For the list of available parameters for logging resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/logging_types.md" %}}.

You can also customize the `fluentd` statefulset that Logging operator deploys. For a list of parameters, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentd_types.md" %}}. For examples on customizing the Fluentd configuration, see {{% xref "/docs/one-eye/logging-operator/configuration/fluentd.md" %}}.

You can also customize the `fluent-bit` that Logging operator deploys. For a list of parameters, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentbit_types.md" %}}. For examples on customizing the Fluent-bit configuration, see {{% xref "/docs/one-eye/logging-operator/configuration/fluentbit.md" %}}.

The following example snippets use the **logging** namespace. To create this namespace if it does not already exist, run:

```bash
kubectl create ns logging
```

## A simple `logging` example {#examples-logging}

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
  namespace: logging
spec:
  fluentd: {}
  fluentbit: {}
  controlNamespace: logging
```

## Filter namespaces

In the following example, the **watchNamespaces** option is set, so logs are collected only from the *prod* and *test* namespaces.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-namespaced
  namespace: logging
spec:
  fluentd: {}
  fluentbit: {}
  controlNamespace: logging
  watchNamespaces: ["prod", "test"]
```
