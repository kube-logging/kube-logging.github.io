---
title: SyslogNGFlow and SyslogNGClusterFlow
weight: 160
---

`SyslogNGFlow` defines a logging flow for **syslog-ng** with **filters** and **outputs**.

The Flow is a `namespaced` resource, so only logs from the same namespaces are collected. You can specify `match` statements to select or exclude logs according to Kubernetes `labels`, container and host names. (Match statements are evaluated in the order they are defined and processed only until the first matching `select` or `exclude` rule applies.) For detailed examples on using the match statement, see [log routing]({{< relref "/docs/logging-operator/configuration/log-routing.md" >}}).

You can define one or more `filters` within a Flow. Filters can perform various actions on the logs, for example, add additional data, transform the logs, or parse values from the records.
The filters in the flow are applied in the order in the definition. You can find the [list of supported filters here]({{< relref "/docs/logging-operator/configuration/plugins/filters">}}).

At the end of the Flow, you can attach one or more [outputs]({{< relref "/docs/logging-operator/configuration/output.md" >}}), which may also be `Output` or `ClusterOutput` resources.

> `SyslogNGFlow` resources are `namespaced`, the `selector` only select `Pod` logs within namespace.
> `SyslogNGClusterFlow` defines a SyslogNGFlow **without** namespace restrictions. It is also only effective in the `controlNamespace`.
 `SyslogNGClusterFlow` selects logs from **ALL** namespace.

The following example selects only messages sent by the log-generator application and forwards them to a syslog output.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: TestFlow
  namespace: default
spec:
  match:
    and:
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/instance
        pattern: one-eye-log-generator
        type: string
    - regexp:
        value:  json.kubernetes.labels.app.kubernetes.io/name
        pattern: log-generator
        type: string
  localOutputRefs:
    - syslog-output
```

- For the details of the `SyslogNGFlow` custom resource, see {{% xref "/docs/logging-operator/configuration/crds/v1beta1/syslogng_flow_types.md" %}}.
- For the details of the `SyslogNGClusterFlow` custom resource, see {{% xref "/docs/logging-operator/configuration/crds/v1beta1/syslogng_clusterflow_types.md" %}}.
- For details on selecting messages, see {{% xref "/docs/logging-operator/configuration/log-routing.md" %}}
- See the [list of supported filters]({{% xref "/docs/logging-operator/configuration/plugins/syslog-ng-filters/_index.md" %}}).
