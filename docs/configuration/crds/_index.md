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

#### Fluent-bit Spec

| Name                    | Type           | Default | Description                                                             |
|-------------------------|----------------|---------|-------------------------------------------------------------------------|
| annotations | map[string]string | {} | Extra annotations to Kubernetes resource|
| labels | map[string]string | {} | Extra labels for fluent-bit and it's related resources |
| tls | [TLS](#tls-spec) | {} | Configure TLS settings|
| image | [ImageSpec](#image-spec) | {} | Fluentd image override |
| resources | [ResourceRequirements](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#resourcerequirements-v1-core) | {} | Resource requirements and limits |
| targetHost | string | *Fluentd host* | Hostname to send the logs forward |
| targetPort | int | *Fluentd port* |  Port to send the logs forward |
| parser | string | cri | Change fluent-bit input parse configuration. [Available parsers](https://github.com/fluent/fluent-bit/blob/master/conf/parsers.conf)  |
| tolerations | [Toleration](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#toleration-v1-core) | {} | Pod toleration |
| metrics | [Metrics]({{< relref "docs/one-eye/logging-operator/operation/logging-operator-monitoring.md#metrics-variables" >}}) | {} | Metrics defines the service monitor endpoints |
| security | [Security]({{< relref "docs/one-eye/logging-operator/configuration/security/_index.md#security-variables" >}}) | {} | Security defines Fluentd, Fluentbit deployment security properties |
| positiondb |  [KubernetesStorage](#kubernetesstorage) | nil | Add position db storage support. If nothing is configured an emptyDir volume will be used. |
| inputTail | [InputTail]({{< relref "docs/one-eye/logging-operator/configuration/fluentbit.md#tail-inputtail" >}}) | {} | Preconfigured tailer for container logs on the host. Container runtime (containerd vs. docker) is automatically detected for convenience. |
| filterKubernetes | [FilterKubernetes]({{< relref "docs/one-eye/logging-operator/configuration/fluentbit.md#kubernetes-filterkubernetes" >}}) | {} | Fluent Bit Kubernetes Filter allows to enrich your log files with Kubernetes metadata. |
| bufferStorage | [BufferStorage]({{< relref "docs/one-eye/logging-operator/configuration/fluentbit.md#bufferstorage" >}}) |  | Buffer Storage configures persistent buffer to avoid losing data in case of a failure |
| bufferStorageVolume | [KubernetesStorage](#kubernetesstorage) | nil | Volume definition for the Buffer Storage. If nothing is configured an emptydir volume will be used. |
| extraVolumeMounts | [][VolumeMount](#volume-mount) | "" | ExtraVolumeMounts defines source and destination foldersof a pod mount |
| customConfigSecret | string | "" | Custom secret to use as fluent-bit config.<br /> It must include all the config files necessary to run fluent-bit (_fluent-bit.conf_, _parsers*.conf_) |
| podPriorityClassName    | string         | ""      | Name of a priority class to launch fluentbit with                       |
| livenessProbe | [Probe](#probe) | {} | Periodic probe of fluentbit container liveness. Container will be restarted if the probe fails. |
| readinessProbe | [Probe](#probe) | {} | Periodic probe of fluentbit container service readiness. Container will be removed from service endpoints if the probe fails. |

**`logging` with custom fluent-bit annotations**

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  fluentd: {}
  fluentbit:
    annotations:
      my-annotations/enable: true
  controlNamespace: logging
```

**`logging` with hostPath volumes for buffers and positions**

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  fluentd: {}
  fluentbit:
    bufferStorageVolume:
      hostPath:
        path: "" # leave it empty to automatically generate
    positiondb:
      hostPath:
        path: "" # leave it empty to automatically generate
  controlNamespace: logging
```

#### Image Spec

Override default images

| Name                    | Type           | Default | Description |
|-------------------------|----------------|---------|-------------|
| repository | string | "" | Image repository |
| tag | string | "" | Image tag |
| pullPolicy | string | "" | Always, IfNotPresent, Never |

**`logging` with custom fluentd image**

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  fluentd: 
    image:
      repository: banzaicloud/fluentd
      tag: v1.10.4-alpine-1
      pullPolicy: IfNotPresent
  fluentbit: {}
  controlNamespace: logging
```





#### Volume Mount

Defines a pod volume mount

| Name                    | Type           | Default | Description |
|-------------------------|----------------|---------|-------------|
| source | string | "" | Source directory to mount |
| destination | string | "" | Destination directory to mount to |
| readOnly | bool | false | Whether the mount is read-only or not |

**`logging` setup with extra volume mount**

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-tls
spec:
  fluentd: {}
  fluentbit:
    extraVolumeMounts:
    - source: /opt/docker
      destination: /opt/docker
      readOnly: true
  controlNamespace: logging
```

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
