---
title: Custom Resource Definitions
shorttitle: CRDs
weight: 300
aliases:
    - /docs/one-eye/logging-operator/crds/
---

This document contains detailed information about the Custom Resource Definitions that the Logging operator uses.

> You can find [example yamls in our GitHub repository](https://github.com/banzaicloud/logging-operator/tree/master/config/samples).

## Namespace separation

A `logging pipeline` consist of two types of resources.

- `Namespaced` resources: `Flow`, `Output`, `SyslogNGFlow`, `SyslogNGOutput`
- `Global` resources: `ClusterFlow`, `ClusterOutput`, `SyslogNGClusterFlow`, `SyslogNGClusterOutput`

The `namespaced` resources are only effective in their **own** namespace. `Global` resources are **cluster wide**.

> You can create `ClusterFlow`, `ClusterOutput`, `SyslogNGClusterFlow`, and `SyslogNGClusterOutput` resources only in the `controlNamespace`, unless the [`allowClusterResourcesFromAllNamespaces`]({{< relref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/logging_types.md#loggingspec-allowclusterresourcesfromallnamespaces" >}}) option is enabled in the logging resource. This namespace **MUST** be a **protected** namespace so that only **administrators** can access it.

## Available CRDs

{{< toc >}}
