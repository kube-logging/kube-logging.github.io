---
title: Fluent Bit log collector
weight: 400
aliases:
    - /docs/one-eye/logging-operator/fluentbit/
    - /docs/one-eye/logging-operator/configuration/fluentbit/
---

<p align="center"><img src="../../img/fluentbit.png" width="340"></p>

Fluent Bit is an open source and multi-platform Log Processor and Forwarder which allows you to collect data/logs from different sources, unify and send them to multiple destinations.

Logging operator uses Fluent Bit as a log collector agent: Logging operator deploys Fluent Bit to your Kubernetes nodes where it collects and enriches the local logs and transfers them to a log forwarder instance.

You can configure the Fluent Bit deployment via the **fluentbit** section of the {{% xref "/docs/one-eye/logging-operator/logging-infrastructure/logging.md" %}}. This page shows some examples on configuring Fluent Bit. For the detailed list of available parameters, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/fluentbit_types.md" %}}.

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

Buffering in Fluent Bit places the processed data into a temporal location until is sent to Fluentd. By default, the Logging operator sets `storage.path` to `/buffers` and leaves fluent-bit defaults for the other options.

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

## Custom Fluent Bit image

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

## Custom Fluent Bit annotations

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

{{< include-headless "cpu-memory-requirements.md" "one-eye/logging-operator" >}}

## Probe

A [Probe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) is a diagnostic performed periodically by the kubelet on a Container. To perform a diagnostic, the kubelet calls a Handler implemented by the Container. You can configure a probe for Fluent Bit in the **livenessProbe** section of the {{% xref "/docs/one-eye/logging-operator/logging-infrastructure/logging.md" %}}. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  fluentbit:
    livenessProbe:
      periodSeconds: 60
      initialDelaySeconds: 600
      exec:
        command:
        - "/bin/sh"
        - "-c"
        - >
          LIVENESS_THRESHOLD_SECONDS=${LIVENESS_THRESHOLD_SECONDS:-300};
          if [ ! -e /buffers ];
          then
            exit 1;
          fi;
          touch -d "${LIVENESS_THRESHOLD_SECONDS} seconds ago" /tmp/marker-liveness;
          if [ -z "$(find /buffers -type d -newer /tmp/marker-liveness -print -quit)" ];
          then
            exit 1;
          fi;
  controlNamespace: logging
```

You can use the following parameters:

| Name                    | Type           | Default | Description |
|-------------------------|----------------|---------|-------------|
| initialDelaySeconds | int | 10 | Number of seconds after the container has started before liveness probes are initiated. |
| timeoutSeconds | int | 0 | Number of seconds after which the probe times out. |
| periodSeconds | int | 10 | How often (in seconds) to perform the probe. |
| successThreshold | int | 0 | Minimum consecutive successes for the probe to be considered successful after having failed. |
| failureThreshold | int | 3 |  Minimum consecutive failures for the probe to be considered failed after having succeeded. |
| exec | array | {} |  Exec specifies the action to take. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/#execaction-v1-core) |
| httpGet | array | {} |  HTTPGet specifies the http request to perform. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/#httpgetaction-v1-core) |
| tcpSocket | array | {} |  TCPSocket specifies an action involving a TCP port. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/#tcpsocketaction-v1-core) |

> Note: To configure readiness probes, see {{% xref "/docs/one-eye/logging-operator/operation/readiness-probe.md" %}}.
