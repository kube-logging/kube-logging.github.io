---
title: Flow and ClusterFlow
weight: 100
---

`Flow` defines a logging flow with **filters** and **outputs**.

The Flow is a `namespaced` resource, so only logs from the same namespaces are collected. You can specify `match` statements to select or exclude logs according to Kubernetes `labels`, container and host names. (Match statements are evaluated in the order they are defined and processed only until the first matching `select` or `exclude` rule applies.) For detailed examples on using the match statement, see [log routing]({{< relref "/docs/one-eye/logging-operator/configuration/log-routing.md" >}}).

You can define one or more `filters` within a Flow. Filters can perform various actions on the logs, for example, add additional data, transform the logs, or parse values from the records.
The filters in the flow are applied in the order in the definition. You can find the [list of supported filters here]({{< relref "/docs/one-eye/logging-operator/configuration/plugins/filters">}}).

At the end of the Flow, you can attach one or more [outputs]({{< relref "/docs/one-eye/logging-operator/configuration/output.md" >}}), which may also be `Output` or `ClusterOutput` resources.

> `Flow` resources are `namespaced`, the `selector` only select `Pod` logs within namespace.
> `ClusterFlow` defines a Flow **without** namespace restrictions. It is also only effective in the `controlNamespace`.
 `ClusterFlow` selects logs from **ALL** namespace.

The following example transforms the log messages from the `default` namespace and sends them to an S3 output.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: flow-sample
  namespace: default
spec:
  filters:
    - parser:
        remove_key_name_field: true
        parse:
          type: nginx
    - tag_normaliser:
        format: ${namespace_name}.${pod_name}.${container_name}
  localOutputRefs:
    - s3-output
  match:
    - select:
        labels:
          app: nginx
```

> Note: In a multi-cluster setup you cannot easily determine which cluster the logs come from. You can append your own labels to each log
using the [record modifier filter](/docs/one-eye/logging-operator/configuration/plugins/filters/record_modifier/).

- For the details of `Flow` custom resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/flow_types.md" %}}.
- For the details of `ClusterFlow` custom resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/clusterflow_types.md" %}}.
- For details on selecting messages, see {{% xref "/docs/one-eye/logging-operator/configuration/log-routing.md" %}}
- See the [list of supported filters]({{< relref "/docs/one-eye/logging-operator/configuration/plugins/filters">}}).
