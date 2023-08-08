---
title: The Logging custom resource
weight: 100
aliases:
    - /docs/one-eye/logging-operator/configuration/logging/
---

The `logging` resource defines the logging infrastructure for your cluster that collects and transports your log messages, and also contains configurations for the Fluent Bit log collector and the Fluentd and syslog-ng log forwarders. It also establishes the `controlNamespace`, the administrative namespace of the Logging operator. The Fluentd and syslog-ng statefulsets and the Fluent Bit daemonset are deployed in this namespace, and global resources like `ClusterOutput` and `ClusterFlow` are evaluated only in this namespace by default - they are ignored in any other namespace unless `allowClusterResourcesFromAllNamespaces` is set to true.

You can customize the configuration of Fluentd, syslog-ng, and Fluent Bit in the logging resource. The logging resource also declares `watchNamespaces`, that specifies the namespaces where `Flow`/`SyslogNGFlow` and `Output`/`SyslogNGOutput` resources will be applied into Fluentd's/syslog-ng's configuration.

{{< include-headless "note-helm-chart-logging-resource.md" >}}

You can customize the following sections of the logging resource:

- Generic parameters of the logging resource. For the list of available parameters, see {{% xref "/docs/configuration/crds/v1beta1/logging_types.md" %}}.
- The `fluentd` statefulset that Logging operator deploys. For a list of parameters, see {{% xref "/docs/configuration/crds/v1beta1/fluentd_types.md" %}}. For examples on customizing the Fluentd configuration, see {{% xref "/docs/logging-infrastructure/fluentd.md" %}}.
- The `syslogNG` statefulset that Logging operator deploys. For a list of parameters, see {{% xref "/docs/configuration/crds/v1beta1/syslogng_types.md" %}}. For examples on customizing the Fluentd configuration, see {{% xref "/docs/logging-infrastructure/syslog-ng.md" %}}.
- The `fluentbit` field is deprecated. Fluent Bit should now be configured separately, see {{% xref "/docs/logging-infrastructure/fluentbit.md" %}}.

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

Starting with Logging operator version 4.3, you can use the `watchNamespaceSelector` selector to select the watched namespaces based on their label, or an expression, for example:

```yaml
  watchNamespaceSelector:
    matchLabels:
      <label-name>: <label-value>
```

```yaml
  watchNamespaceSelector:
    matchExpressions:
      - key: "<label-name>"
        operator: NotIn
        values:
          - "<label-value>"
```

If both `watchNamespaces` and `watchNamespaceSelector` are set, the union of them will take effect.
