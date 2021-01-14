---
title: Custom Resource Definitions
shorttitle: CRDs
weight: 300
aliases:
    - /docs/one-eye/logging-operator/crds/
---



This document contains the detailed information about the CRDs Logging operator uses.

Available CRDs:

- [loggings.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_loggings.yaml)
- [outputs.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_outputs.yaml)
- [flows.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_flows.yaml)
- [clusteroutputs.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_clusteroutputs.yaml)
- [clusterflows.logging.banzaicloud.io](https://github.com/banzaicloud/logging-operator/tree/master/config/crd/bases/logging.banzaicloud.io_clusterflows.yaml)

> You can find [example yamls in our GitHub repository](https://github.com/banzaicloud/logging-operator/tree/master/config/samples).

## Namespace separation

> The [One Eye](/products/one-eye/) observability tool can [manage the TLS certificates of the logging resource](/docs/one-eye/tls/) using cert-manager.

### Namespace separation

A `logging pipeline` consist two type of resources.

- `Namespaced` resources: `Flow`, `Output`
- `Global` resources: `ClusterFlow`, `ClusterOutput`

The `namespaced` resources only effective in their **own** namespace. `Global` resources are operate **cluster wide**.

> You can only create `ClusterFlow` and `ClusterOutput` in the `controlNamespace`. It **MUST** be a **protected** namespace that only **administrators** have access.


The `namespaced` resources are only effective in their **own** namespace. `Global` resources are **cluster wide**.

> You can create `ClusterFlow` and `ClusterOutput` resources only in the `controlNamespace`. This namespace **MUST** be a **protected** namespace so that only **administrators** can access it.

#### KubernetesStorage

Define Kubernetes storage

| Name      | Type | Default | Description |
|-----------|------|---------|-------------|
| hostPath | [HostPathVolumeSource](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#hostpathvolumesource-v1-core) | - | Represents a host path mapped into a pod. If path is empty, it will automatically be set to "/opt/logging-operator/<name of the logging CR>/<name of the volume>" |
| emptyDir | [EmptyDirVolumeSource](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#emptydirvolumesource-v1-core) | - | Represents an empty directory for a pod. |
| pvc | [PersistentVolumeClaim](#persistent-volume-claim) | - | A PersistentVolumeClaim (PVC) is a request for storage by a user. |

#### Persistent Volume Claim

| Name      | Type | Default | Description |
|-----------|------|---------|-------------|
| spec | [PersistentVolumeClaimSpec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#persistentvolumeclaimspec-v1-core) | - | Spec defines the desired characteristics of a volume requested by a pod author. |
| source | [PersistentVolumeClaimVolumeSource](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#persistentvolumeclaimvolumesource-v1-core) | - | PersistentVolumeClaimVolumeSource references the user's PVC in the same namespace.  |

The Persistent Volume Claim should be created with the given `spec` and with the `name` defined in the `source`'s `claimName`.
