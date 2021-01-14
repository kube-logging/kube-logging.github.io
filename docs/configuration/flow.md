---
title: Flow and ClusterFlow
weight: 100
---

Flows define a `logging flow` that defines the `filters` and `outputs`.
`Flow` defines a logging flow with **filters** and **outputs**. This is a `namespaced` resource as well, so only logs from the same namespaces are collected. You can specify `selectors` to filter logs according to Kubernetes `labels`, and can define one or more `filters` within a Flow. These filters are applied in the order in the definition. You can find the supported filters list, [here]({{< relref "/docs/one-eye/logging-operator/configuration/plugins/filters">}}). At the end of the Flow, you can attach one or more outputs, which may also be `Output` or `ClusterOutput` resources.

> `Flow` resources are `namespaced`, the `selector` only select `Pod` logs within namespace.
> `ClusterFlow` defines a Flow **without** namespace restrictions. It is also only effective in the `controlNamespace`.
 `ClusterFlow` select logs from **ALL** namespace.

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

- For the details of `Flow` custom resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/flow_types.md" %}}.
- For the details of `ClusterFlow` custom resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/clusterflow_types.md" %}}.
- For details on selecting messages, see {{% xref "/docs/one-eye/logging-operator/configuration/log-routing.md" %}}
