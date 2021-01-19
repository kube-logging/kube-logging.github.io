---
title: Custom Resource Definitions
shorttitle: CRDs
weight: 300
aliases:
    - /docs/one-eye/logging-operator/crds/
---

This document contains detailed information about the Custom Resource Definitions that the Logging operator uses.

Available CRDs:

- [loggings.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_loggings.yaml)
- [outputs.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_outputs.yaml)
- [flows.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_flows.yaml)
- [clusteroutputs.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_clusteroutputs.yaml)
- [clusterflows.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_clusterflows.yaml)

> You can find [example yamls in our GitHub repository](https://github.com/banzaicloud/logging-operator/tree/master/config/samples).

## Namespace separation

A `logging pipeline` consist of two types of resources.

- `Namespaced` resources: `Flow`, `Output`
- `Global` resources: `ClusterFlow`, `ClusterOutput`

The `namespaced` resources are only effective in their **own** namespace. `Global` resources are **cluster wide**.

> You can create `ClusterFlow` and `ClusterOutput` resources only in the `controlNamespace`. This namespace **MUST** be a **protected** namespace so that only **administrators** can access it.
