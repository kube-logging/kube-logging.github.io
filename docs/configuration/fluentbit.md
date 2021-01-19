---
title: FluentBit
weight: 600
aliases:
    - /docs/one-eye/logging-operator/fluentbit/
---

<p align="center"><img src="../../img/fluentbit.png" width="340"></p>

Fluent Bit is an open source and multi-platform Log Processor and Forwarder which allows you to collect data/logs from different sources, unify and send them to multiple destinations.

You can configure the Fluent-bit deployment via the **fluentbit** section of the {{% xref "/docs/one-eye/logging-operator/configuration/logging.md" %}}. This page shows some examples on configuring Fluent-bit. For the detailed list of available parameters, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentbit_types.md" %}}.

## Filters

### Kubernetes (filterKubernetes)

Fluent Bit Kubernetes Filter allows you to enrich your log files with Kubernetes metadata. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  fluentd: {}
  fluentbit:
    filterKubernetes:
       Kube_URL: "https://kubernetes.default.svc:443"
       Match: "kube.*"
  controlNamespace: logging
```

For the detailed list of available parameters for this plugin, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentbit_types.md#filterkubernetes" %}}.
[More info](https://github.com/fluent/fluent-bit-docs/blob/master/filter/kubernetes.md)

## Tail input

The tail input plugin allows to monitor one or several text files. It has a similar behavior like *tail -f* shell command. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  fluentd: {}
  fluentbit:
    inputTail:
       Refresh_Interval: "60"
       Rotate_Wait: "5"
  controlNamespace: logging
```

For the detailed list of available parameters for this plugin, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentbit_types.md#inputtail" %}}.
[More Info](https://github.com/fluent/fluent-bit-docs/blob/1.3/input/tail.md).

## Buffering

Buffering in Fluent-bit places the processed data into a temporal location until is sent to Fluentd. By default, the Logging operator sets `storage.path` to `/buffers` and leaves fluent-bit defaults for the other options.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  fluentd: {}
  fluentbit:
    bufferStorage:
       storage.path: /buffers
  controlNamespace: logging
```

For the detailed list of available parameters for this plugin, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentbit_types.md#bufferstorage" %}}.
[More Info](https://docs.fluentbit.io/manual/v/1.3/configuration/buffering).

### HostPath volumes for buffers and positions

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

## Custom Fluent-bit image

You can deploy custom images by overriding the default images using the following parameters in the fluentd or fluentbit sections of the logging resource.

The following example deploys a custom fluentd image:

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

## Volume Mount

Defines a pod volume mount. For example:

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

For the detailed list of available parameters for this plugin, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentbit_types.md#volumemount" %}}.

## Custom Fluent-bit annotations

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

## KubernetesStorage

Define Kubernetes storage.

| Name      | Type | Default | Description |
|-----------|------|---------|-------------|
| hostPath | [HostPathVolumeSource](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#hostpathvolumesource-v1-core) | - | Represents a host path mapped into a pod. If path is empty, it will automatically be set to "/opt/logging-operator/<name of the logging CR>/<name of the volume>" |
| emptyDir | [EmptyDirVolumeSource](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#emptydirvolumesource-v1-core) | - | Represents an empty directory for a pod. |
