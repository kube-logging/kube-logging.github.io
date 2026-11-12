---
title: Fluent Bit log collector
weight: 400
aliases:
    - /docs/one-eye/logging-operator/fluentbit/
    - /docs/one-eye/logging-operator/configuration/fluentbit/
---

<p align="center"><img src="../../img/fluentbit.png" alt="Logos" width="340"></p>

Fluent Bit is an open source and multi-platform Log Processor and Forwarder which allows you to collect data/logs from different sources, unify and send them to multiple destinations.

Logging operator uses Fluent Bit as a log collector agent: Logging operator deploys Fluent Bit to your Kubernetes nodes where it collects and enriches the local logs and transfers them to a log forwarder instance.

## Ways to configure Fluent Bit

There are three ways to configure the Fluent Bit daemonset:

1. Using the **spec.fluentbit** section of {{% xref "/docs/logging-infrastructure/logging.md" %}}. This method is deprecated and will be removed in the next major release.
1. Using the standalone FluentbitAgent CRD. This method is only available in Logging operator version 4.2 and newer, and the specification of the CRD is compatible with the **spec.fluentbit** configuration method.
1. Using the **spec.nodeagents** section of {{% xref "/docs/logging-infrastructure/logging.md" %}}. This method is deprecated and will be removed from the Logging operator. (Note that this configuration isn't compatible with the FluentbitAgent CRD.)

For the detailed list of available parameters, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md" %}}.

### Migrating from **spec.fluentbit** to FluentbitAgent {#migrating}

The standalone FluentbitAgent CRD is only available in Logging operator version 4.2 and newer. Its specification and logic is identical with the **spec.fluentbit** configuration method. Using the FluentbitAgent CRD allows you to remove the **spec.fluentbit** section from the Logging CRD, which has the following benefits.

- RBAC control over the FluentbitAgent CRD, so you can have separate roles that can manage the Logging resource and the FluentbitAgent resource (that is, the Fluent Bit deployment).
- It reduces the size of the Logging resource, which can grow big enough to reach the annotation size limit in certain scenarios (e.g. when using `kubectl apply`).
- It allows you to use multiple different Fluent Bit configurations within the same cluster. For details, see {{% xref "/docs/logging-infrastructure/fluentbit-multiple.md" %}}.

To migrate your **spec.fluentbit** configuration from the Logging resource to a separate FluentbitAgent CRD, complete the following steps.

1. Open your Logging resource and find the **spec.fluentbit** section. For example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: example-logging-resource
    spec:
        controlNamespace: default
        fluentbit:
            inputTail:
              storage.type: filesystem
            positiondb:
              hostPath:
                path: ""
            bufferStorageVolume:
              hostPath:
                path: ""
    ```

1. Create a new FluentbitAgent CRD. For the value of **metadata.name**, use the name of the Logging resource, for example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: FluentbitAgent
    metadata:
      # Use the name of the logging resource
      name: example-logging-resource
    ```

1. Copy the the **spec.fluentbit** section from the Logging resource into the **spec** section of the FluentbitAgent CRD, then fix the indentation.

1. Specify the paths for the positiondb and the bufferStorageVolume. If you used the default settings in the **spec.fluentbit** configuration, set empty strings as paths, like in the following example. This is needed to retain the existing buffers of the deployment, otherwise data loss may occur.

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: FluentbitAgent
    metadata:
      # Use the name of the logging resource
      name: example-logging-resource
    spec:
      inputTail:
        storage.type: filesystem
      positiondb:
        hostPath:
          path: ""
      bufferStorageVolume:
        hostPath:
          path: ""
    ```

1. Delete the **spec.fluentbit** section from the Logging resource, then apply the Logging and the FluentbitAgent CRDs.

<!-- FIXME add a step on how to check that everything is working, for example, how to check the ownership?
The ownerrefs of the managed resources changed from the Logging resource to the new FluentbitAgent resource.
-->

## Examples

The following sections show you some examples on configuring Fluent Bit. For the detailed list of available parameters, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md" %}}.

> Note: These examples use the traditional method that configures the Fluent Bit deployment using **spec.fluentbit** section of {{% xref "/docs/logging-infrastructure/logging.md" %}}.

## Containerd log fields

The following example defines a custom Fluent Bit parser that places the parsed containerd log messages into the `log` field instead of the `message` field to be backwards compatible with docker container runtimes.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: containerd
spec:
  inputTail:
    Parser: cri-log-key
  # Parser that populates `log` instead of `message` to enable the Kubernetes filter's Merge_Log feature to work
  # Mind the indentation, otherwise Fluent Bit will parse the whole message into the `log` key
  customParsers: |
                  [PARSER]
                      Name cri-log-key
                      Format regex
                      Regex ^(?<time>[^ ]+) (?<stream>stdout|stderr) (?<logtag>[^ ]*) (?<log>.*)$
                      Time_Key    time
                      Time_Format %Y-%m-%dT%H:%M:%S.%L%z
  # Required key remap if one wants to rely on the existing auto-detected log key in the fluentd parser and concat filter otherwise should be omitted
  filterModify:
    - rules:
      - Rename:
          key: log
          value: message
  # Configure filter grep for fluentbit
  filterGrep:
    Match: "*"
    Exclude:
      - message ^.\"level\"\:\"(debug|DEBUG)\".$
```

## Filters

### Kubernetes (filterKubernetes)

Fluent Bit Kubernetes Filter allows you to enrich your log files with Kubernetes metadata. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default
spec:
  filterKubernetes:
    Kube_URL: "https://kubernetes.default.svc:443"
```

For the detailed list of available parameters for this plugin, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md#filterkubernetes" %}}.
[More info](https://docs.fluentbit.io/manual/pipeline/filters/kubernetes)

### Multiline exceptions (filterMultiline) {#filtermultiline}

Use the Fluent Bit multiline filter to combine stack traces, such as Java or Python exceptions, into a single record. Fluent Bit combines them on the node, before the logs reach Fluentd. This is an alternative to the Fluentd [`detectExceptions`]({{< relref "/docs/configuration/plugins/filters/detect_exceptions.md" >}}) filter, which doesn't support multiple Fluentd workers. After moving exception detection to Fluent Bit, you can remove `detectExceptions` from your flows and run Fluentd with more than one worker. Every other Fluentd plugin you use must also support multiple workers.

The filter is disabled by default. When you configure `filterMultiline`, the Logging operator places it before the other Fluent Bit filters. Fluent Bit re-emits the combined records to the start of the pipeline, so any filter placed before the multiline filter would process them twice.

The following example assumes CRI container logs. The `cri` parser of the tail input first reassembles log lines that the container runtime split into fragments. Then the filter applies the built-in `java` parser and a custom `python_exception` parser defined in `customParsers`.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: multiline
spec:
  inputTail:
    multiline.parser:
      - cri
  customParsers: |
    [MULTILINE_PARSER]
        Name python_exception
        Type regex
        Flush_Timeout 2000
        Key_Content log
        Key_Group stream
        Rule "start_state" "/^Traceback \(most recent call last\):$/" "python"
        Rule "python" "/^[\t ]+.+$/" "python"
        Rule "python" "/^(?:[^\s.():]+\.)*[^\s.():]+:.*$/" "python_end"
        Rule "python" "/^(?:KeyboardInterrupt|SystemExit|StopIteration|GeneratorExit)$/" "python_end"
        Rule "python_end" "/^\s*$/" "python_end"
        Rule "python_end" "/^(?:During handling of the above exception, another exception occurred:|The above exception was the direct cause of the following exception:)$/" "python_chain"
        Rule "python_chain" "/^\s*$/" "python_chain"
        Rule "python_chain" "/^Traceback \(most recent call last\):$/" "python"
  filterMultiline:
    match: kubernetes.*
    # Comma-separated list of built-in or custom multiline parsers
    parser: java, python_exception
    keyContent: log
    buffer: "On"
    flushMs: 2000
    emitterName: exception_multiline
```

To run Fluentd with multiple workers, set `workers` in the `fluentd` section of your `logging` resource, for example, `workers: 2`. For details, see {{% xref "/docs/configuration/crds/v1beta1/fluentd_types.md#fluentdspec-workers" %}}.

With buffering enabled, the filter holds incomplete records in memory until it emits them. To change where and how much it buffers, set `emitterStorageType` (`memory` or `filesystem`) and `emitterMemBufLimit`. If your container runtime isn't CRI-based, replace the `cri` tail parser with one that matches your runtime.

For the detailed list of available parameters, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md" %}}.
[More info](https://docs.fluentbit.io/manual/data-pipeline/filters/multiline-stacktrace)

## Tail input

The tail input plugin allows to monitor one or several text files. It has a similar behavior like *tail -f* shell command. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  inputTail:
    storage.type: filesystem
    Refresh_Interval: "60"
    Rotate_Wait: "5"
```

For the detailed list of available parameters for this plugin, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md#inputtail" %}}.
[More Info](https://github.com/fluent/fluent-bit-docs/blob/1.3/input/tail.md).

## Buffering

Buffering in Fluent Bit places the processed data into a temporal location until is sent to Fluentd. By default, the Logging operator sets `storage.path` to `/buffers` and leaves fluent-bit defaults for the other options.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  inputTail:
    storage.type: filesystem
  bufferStorage:
    storage.path: /buffers
```

For the detailed list of available parameters for this plugin, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md#bufferstorage" %}}.
[More Info](https://docs.fluentbit.io/manual/v/1.3/configuration/buffering).

### Flush the filesystem buffer on shutdown

Setting `storage.backlog.flush_on_shutdown` to `"On"` makes Fluent Bit flush all filesystem-backlog chunks to their destination during a graceful shutdown. This way, you don't lose buffered records when a node is drained or a spot instance is reclaimed.

This option is opt-in and defaults to `"Off"`. The operator renders the option only if you set it.

Enable filesystem buffering first, or this option has no effect: set `storage.type: filesystem` on the tail input and set `storage.path`. This setting only affects filesystem-backlog chunks.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  inputTail:
    storage.type: filesystem
  bufferStorage:
    storage.path: /buffers
    storage.backlog.flush_on_shutdown: "On"
```

Increase [`terminationGracePeriodSeconds`]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md#fluentbitspec-terminationgraceperiodseconds" >}}) so the pod has enough time to finish flushing before Kubernetes force-kills it.

### HostPath volumes for buffers and positions

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  inputTail:
    storage.type: filesystem
  bufferStorageVolume:
    hostPath:
      path: "" # leave it empty to automatically generate
  positiondb:
    hostPath:
      path: "" # leave it empty to automatically generate
```

## Custom Fluent Bit image

You can deploy custom images by overriding the default images using the following parameters.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  image:
    repository: fluent/fluent-bit
    tag: 2.1.8-debug
    pullPolicy: IfNotPresent
```

## Log mount path

If your Kubernetes cloud provider or environment saves the container logs in a directory other than the default, set the `mountPath` field to the path where the logs are saved. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  mountPath: /data/docker/containers
```

For details, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md#fluentbitspec-mountpath" %}}.

## Volume Mount

Defines a pod volume mount. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging
spec:
  extraVolumeMounts:
  - destination: /data/docker/containers
    readOnly: true
    source: /data/docker/containers
```

For the detailed list of available parameters for this plugin, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md#volumemount" %}}.

## Custom Fluent Bit annotations

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  annotations:
    my-annotations/enable: true
```

## KubernetesStorage

Define Kubernetes storage.

| Name      | Type | Default | Description |
|-----------|------|---------|-------------|
| hostPath | [HostPathVolumeSource](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#hostpathvolumesource-v1-core) | - | Represents a host path mapped into a pod. If path is empty, it will automatically be set to `/opt/logging-operator/<name of the logging CR>/<name of the volume>` |
| emptyDir | [EmptyDirVolumeSource](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#emptydirvolumesource-v1-core) | - | Represents an empty directory for a pod. |

{{< include-headless "cpu-memory-requirements.md" >}}

## Probe

A [Probe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) is a diagnostic performed periodically by the kubelet on a Container. To perform a diagnostic, the kubelet calls a Handler implemented by the Container. You can configure a probe for Fluent Bit in the **livenessProbe** section of the {{% xref "/docs/logging-infrastructure/logging.md" %}}. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: default-logging-simple
spec:
  livenessProbe:
    periodSeconds: 60
    initialDelaySeconds: 600
    exec:
      command:
      - "/bin/sh"
      - "-c"
      - >
        LIVENESS_THRESHOLD_SECONDS=${LIVENESS_THRESHOLD_SECONDS:-300};
        if [ ! -e /buffers ]; then
          exit 1;
        fi;
        touch -d "${LIVENESS_THRESHOLD_SECONDS} seconds ago" /tmp/marker-liveness;
        if [ -z "$(find /buffers -type d -newer /tmp/marker-liveness -print -quit)" ]; then
          exit 1;
        fi;
```

You can use the following parameters:

| Name                    | Type           | Default | Description |
|-------------------------|----------------|---------|-------------|
| initialDelaySeconds | int | 10 | Number of seconds after the container has started before liveness probes are initiated. |
| timeoutSeconds | int | 0 | Number of seconds after which the probe times out. |
| periodSeconds | int | 10 | How often (in seconds) to perform the probe. |
| successThreshold | int | 0 | Minimum consecutive successes for the probe to be considered successful after having failed. |
| failureThreshold | int | 3 |  Minimum consecutive failures for the probe to be considered failed after having succeeded. |
| exec | array | {} |  Exec specifies the action to take. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#execaction-v1-core) |
| httpGet | array | {} |  HTTPGet specifies the http request to perform. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#httpgetaction-v1-core) |
| tcpSocket | array | {} |  TCPSocket specifies an action involving a TCP port. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#tcpsocketaction-v1-core) |

> Note: To configure readiness probes, see {{% xref "/docs/operation/readiness-probe.md" %}}.
