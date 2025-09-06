---
title: What's new
weight: 50
---

## Version 6.0

### Breaking changes

As announced in the 5.4 release notes, the 6.0 release includes the following breaking changes:

- NodeAgent CRD and inline NodeAgents in the Logging resource have been removed. See [migration tips](#nodeagent-migration).
- hostTailer cannot be configured from the Helm chart anymore, [use hosttailers instead](#hosttailer-migration).

For details, see the [deprecation notice for 5.4](#upcoming-deprecations-and-breaking-changes). Note that 5.4.0 remains officially supported until October 6. (If needed, patch releases from community contributions for version 5.4.0 will be supported even after 6th of October.)

## Version 5.4

The following are the highlights and main changes of Logging operator 5.4. For a complete list of changes and bugfixes, see the [Logging operator 5.4 releases page](https://github.com/kube-logging/logging-operator/releases/tag/5.4.0).

- The new [LogicMonitor Logs]({{< relref "/docs/configuration/plugins/outputs/lm_logs.md" >}}) Fluentd output plugin allows you to send logs to [LM Logs of LogicMonitor](https://www.logicmonitor.com/).
- The Elasticsearch Fluent plugin has been updated to 6.0, making [wildcards available in index patterns by default](/docs/configuration/plugins/outputs/elasticsearch.md#elasticsearch-data_stream_template_use_index_patterns_wildcard).
- You can now use the `compress` option in the [forward output]({{< relref "/docs/configuration/plugins/outputs/forward.md#forwardoutput-compress" >}}) to enable gzip compression.
- You can [disable mounting the `/var/log` volume]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md#fluentbitspec-disablevarlog" >}}) in Fluent Bit. This is useful when you're not permitted to mount host volumes, and collect host logs some other way.
- Initial basic implementation of the upcoming [AxoSyslog custom resource]({{< relref "/docs/configuration/crds/v1beta1/axosyslog_types.md" >}}).

### Upcoming deprecations and breaking changes

We are planning on deprecating the following components in the next major release. These breaking changes will be part of version 6.0.0 (scheduled for July 14), while 5.4.0 remains officially supported until October 6. (If needed, patch releases from community contributions for version 5.4.0 will be supported even after 6th of October.)

#### NodeAgent CRD and inline NodeAgents in the Logging resource {#nodeagent-migration}

NodeAgents were an attempt to generalize log agents configuration, but it never got out PoC status, with the main feature of running Fluent Bit on Windows hosts. The code hasn't been updated recently, and the original FluentbitSpec (in the Logging resource and in the separate FluentbitAgent resource) and the features behind it have significantly diverged.

Last year we've introduced [Telemetry Controller](https://github.com/kube-logging/telemetry-controller) as a replacement for the NodeAgent and FluentbitAgent resources, with additional multi-tenant capabilities and more flexible agent-side log selection.

- If you're using NodeAgent on Windows, get in touch with us (the Logging operator maintainers) over the [community channels]({{< relref "/docs/community.md" >}}), so we can help you find a viable path forward using the Telemetry Controller.
- Non-windows users should either migrate to [FluentbitAgent]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md" >}}), or to the [Telemetry Controller](https://github.com/kube-logging/telemetry-controller). In case you need help with either case, [feel free to contact us]({{< relref "/docs/community.md" >}}).

#### hostTailer in the Helm chart {#hosttailer-migration}

Configuring a hostTailer in the [Logging operator Helm chart](https://github.com/kube-logging/logging-operator/tree/master/charts/logging-operator) is deprecated in favor of using hostTailers. Migrate your hostTailer configuration like this:

- Old configuration:

    ```yaml
      hostTailer:
        # -- HostTailer
        enabled: false
        # -- name of HostTailer
        name: hosttailer
        image:
          # -- repository of eventTailer image
          repository:
          # -- tag of eventTailer image
          tag:
          # -- pullPolicy of eventTailer image
          pullPolicy:
          # -- imagePullSecrets of eventTailer image
          imagePullSecrets: []
        # -- workloadMetaOverrides of HostTailer
        workloadMetaOverrides:
        # -- workloadOverrides of HostTailer
        workloadOverrides:
        # -- configure fileTailers of HostTailer
        # example:
        #   - name: sample-file
        #     path: /var/log/sample-file
        #     disabled: false
        #     buffer_max_size:
        #     buffer_chunk_size:
        #     skip_long_lines:
        #     read_from_head: false
        #     containerOverrides:
        #     image:
        fileTailers: []
        # -- configure systemdTailers of HostTailer
        # example:
        #   - name: system-sample
        #     disabled: false
        #     systemdFilter: kubelet.service
        #     maxEntries: 20
        #     containerOverrides:
        #     image:
        systemdTailers: **[]**
    ```

- The new configuration will be similar to this:

    ```yaml
      hostTailers:
        # -- Enable all hostTailers
        enabled: false
        # -- List of hostTailers configurations
        instances: []
        # - name: hosttailer
          # -- Enable hostTailer
          # enabled: true
          # image:
            # -- repository of eventTailer image
            # repository:
            # -- tag of eventTailer image
            # tag:
            # -- pullPolicy of eventTailer image
            # pullPolicy:
            # -- imagePullSecrets of eventTailer image
            # imagePullSecrets: []
          # -- workloadMetaOverrides of HostTailer
          # workloadMetaOverrides: {}
          # -- workloadOverrides of HostTailer
          # workloadOverrides: {}
          # -- configure fileTailers of HostTailer
          # fileTailers:
          # - name: sample-file
          #   path: /var/log/sample-file
          #   disabled: false
          #   buffer_max_size:
          #   buffer_chunk_size:
          #   skip_long_lines:
          #   read_from_head: false
          #   containerOverrides:
          #   image:
          # -- configure systemdTailers of HostTailer
          # systemdTailers:
          # - name: system-sample
          #   disabled: false
          #   systemdFilter: kubelet.service
          #   maxEntries: 20
          #   containerOverrides:
          #   image:
    ```

## Version 5.3

The following are the highlights and main changes of Logging operator 5.3. For a complete list of changes and bugfixes, see the [Logging operator 5.3 releases page](https://github.com/kube-logging/logging-operator/releases/tag/5.3.0).

### Breaking change

Starting with this version, Logging operator sets default security context values for the Fluentd logging configuration, so from now on:

- Fluentd runs as the `fluentd` user (instead of root)
- Default user and group IDs are set in the SecurityContext and PodSecurityContext

### Other changes

- In this release we've moved `fluentd-drain-watch` and `node-exporter` into the Logging operator repository. From now on, we build these images from our repository (https://github.com/kube-logging/logging-operator/tree/master/images/) and their version numbering follows the version numbers of Logging operator.
- You can now set the `format_key` and `format_name` options for when using the [Fluentd parser filter]({{< relref "/docs/configuration/plugins/filters/parser.md" >}}) to use multi-format parsing.
- You can use Filter Grep (https://docs.fluentbit.io/manual/pipeline/filters/grep) in the [FluentbitSpec]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md#filtergrep" >}}) section of your logging configuration. That way you can exclude logs before passing them to Fluentd.
- When using Fluentd and ClusterFlows, you can now select and exclude namespaces with regular expressions using the `namespaces_regex` option.
- When using the [Loki destination]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/loki.md#lokioutput-tenant-id" >}}) in a SyslogNGClusterOutput/SyslogNGOutput, you can now set the tenant-id.
- You can now enable verbose mode in {{% xref "/docs/configuration/crds/extensions/hosttailer_types.md" %}}. Also, the default log level of the HostTailer has been reduced to error.

## Version 5.2

The following are the highlights and main changes of Logging operator 5.2. For a complete list of changes and bugfixes, see the [Logging operator 5.2 releases page](https://github.com/kube-logging/logging-operator/releases/tag/5.2.0).

You can now disable mounting the `/var/lib/docker/containers` volume using the `DisableVarLibDockerContainers` option in the `loggings` or `FluentbitAgent` CRD. This solves an installation error on GKE Autopilot.

In this release we've moved `config-reloader` and `syslog-ng-reloader` into the Logging operator repository. From now on, we build these images from our repository (https://github.com/kube-logging/logging-operator/tree/master/images/) and their version numbering follows the version numbers of Logging operator.

## Version 5.1

The following are the highlights and main changes of Logging operator 5.1. For a complete list of changes and bugfixes, see the [Logging operator 5.1 releases page](https://github.com/kube-logging/logging-operator/releases/tag/5.1.1).

### Fluentd http output improvements

- Set the `compress` option to `gzip` to compress the HTTP request body.
- You can use the `headers_from_placeholders` option to add headers to the HTTP requests.
- Set the `reuse_connection` option to `true` to try to reuse HTTP connections to improve performance.

### Multiple hosttailer support

You can now define multiple hosttailers in your logging configuration, for example:

```yaml
logging:
  enabled: true
  hostTailers:
    enabled: true
    instances:
      - name: kubeauditane
        enabled: true
        workloadOverrides:
          nodeSelector:
            node-role.kubernetes.io/control-plane: "true"
          tolerations:
            - key: node-role.kubernetes.io/control-plane
              operator: Exists
              effect: NoSchedule
        fileTailers:
          - name: kube-audit
            path: /var/lib/rancher/rke2/server/logs/*.log

      - name: workersnodesonly
        enabled: true
        workloadOverrides:
          nodeSelector:
            node-role.kubernetes.io/worker: "true"
        fileTailers:
          - name: kube-audit
            path: /var/lib/rancher/rke2/agent/logs/*.log
```

This also means that `logging.hostTailer` has been deprecated and is superseded by `logging.hostTailers` and will be removed in a future release.

## Graceful reload in Fluentd

You can now enable graceful reloading via a webhook using the [`configReloaderUseGracefulReloadWebhook` option]({{< relref "/docs/configuration/crds/v1beta1/fluentd_types.md#fluentdspec-configreloaderusegracefulreloadwebhook" >}}).

### Memory usage

In order to reduce the memory usage of the operator in large environments, you can now use the following flags during installation:

- `watch-labeled-children` to watch only child resources created by the operator. This option will be enabled by default in a future new minor version.
- `watch-labeled-secrets` to watch secrets with `logging.banzaicloud.io/watch: enabled` label. This option will be enabled by default in a future new major version.

### Changes in Fluentd images

Earlier, Logging operator used Fluentd images published at https://github.com/kube-logging/fluentd-images, and its version numbers followed the official Fluentd version numbers. From now on, we build Fluentd images from our repository (https://github.com/kube-logging/logging-operator/tree/master/images/fluentd) and its version numbering follows the version numbers of Logging operator, for example: ghcr.io/kube-logging/logging-operator/fluentd:5.1.1-base/filters/full

## Version 5.0

The following are the highlights and main changes of Logging operator 5.0. For a complete list of changes and bugfixes, see the [Logging operator 5.0 releases page](https://github.com/kube-logging/logging-operator/releases/tag/5.0).

### Breaking changes

- The Sumo Logic filter, the Sumo Logic output, and the `enhance_k8s` filter are no longer supported, as they are not available in the new Fluentd image (v1.17-5.0). If you want to continue using them, keep using the 4.x version of Logging operator.
- The name of the `crd` subchart changed to `logging-operator-crds`. This new subchart is also available as an OCI artifact.

### Clean up stuck finalizers

When uninstalling Logging operator using Helm, some finalizers may be stuck because Helm uninstalls the resources in a non-deterministic order. You can use the new `finalizer-cleanup` flag in conjunction with `.Values.rbac.retainOnDelete` to avoid this problem. When both options are set during uninstall, then:

- Helm will retain the operator's service account, cluster role, and cluster role binding. This is necessary because Helm uninstalls resources in an unpredictable order.
- The operator will attempt to free up its managed logging resources by removing finalizers, which would otherwise remain stuck.

For example:

```shell
helm install logging-operator ./charts/logging-operator/ --set rbac.retainOnDelete=true --set extraArgs='{"-enable-leader-election=true","-finalizer-cleanup=true"}'
```

### Experimental Telemetry Controller support

You can now use the [Telemetry Controller](https://github.com/kube-logging/telemetry-controller) as a log collector agent instead of Fluent Bit. To test this feature, you have to:

1. Install Logging operator with the `enable-telemetry-controller-route` flag and the `telemetry-controller.install` options enabled:

    ```shell
    helm install logging-operator ./charts/logging-operator/ --set extraArgs='{"-enable-leader-election=true","-enable-telemetry-controller-route"}' --set telemetry-controller.install=true
    ```

1. Configure the `routeConfig` option in the `Logging` resource to specify how the Telemetry Controller resources are created:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: tc-config
    spec:
      routeConfig:
        enableTelemetryControllerRoute: true # Determines whether to create TC resources for log-collection and routing purposes.
        disableLoggingRoute: false # Determines whether to use Logging-routes for routing purposes and Fluentbit-agents for log-collection.
        tenantLabels: # Will be placed on TC's tenant resources, must be matched with the same field on the Collector resource. (Deployed by the cluster administrator.)
          tenant: logging
    ```

**NOTE: There is a hands-on example available to try: <https://github.com/kube-logging/logging-operator/tree/master/config/samples/telemetry-controller-routing>**

### Improved IPv6 support

- Fluent Bit couldn't listen on an IPv6 http socket because its HTTP_Listen address was hardcoded. Now this is set using the `POD_IP`, so it works in IPv6 environments as well.
- Until now, IPv6-only clusters couldn't scrape metrics from the Fluentd aggregator. This has been fixed.

### rdkafka option support

You can now set `rdkafka_options` in for rdkafka2 in the [Kafka Fluentd output]({{< relref "/docs/configuration/plugins/outputs/kafka.md#kafka-rdkafka_options" >}}).

### Other Fluent Bit changes

- You can now specify whether to pause or drop data when the buffer is full. This helps to make sure we apply backpressure on the input. For details, see {{% xref "/docs/configuration/crds/v1beta1/fluentbit_types.md#inputtail-storage.pause_on_chunks_overlimit" %}}.
- HotReload pauses all inputs and waits until they finish. However, this can block the reload indefinitely, for example, if an output is down for a longer time. You can now force the reload to happen after a grace period using the [`forceHotReloadAfterGrace`]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md#fluentbitspec-forcehotreloadaftergrace" >}}) option.

## Version 4.11

The following are the highlights and main changes of Logging operator 4.11. For a complete list of changes and bugfixes, see the [Logging operator 4.11 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.11.0).

- You can now set the `protected` flag for SyslogNGClusterOutput kinds.
- Charts and images are now signed. To verify the signature, see {{% xref "/docs/install/_index.md#verify" %}}.
- You can now add annotations and labels to Persistent Volume Claims of the Fluentd StatefulSet. For example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: all-to-file
    spec:
      controlNamespace: default
      fluentd:
        bufferStorageVolume:
          pvc:
            labels:
              app: logging
            annotations:
              app: logging
            source:
              claimName: manual
              readOnly: false
    ```

- You can now set liveness probes to the buffer-metrics sidecar container using the `bufferVolumeLivenessProbe` option.
- IPv6 improvements:

    - You can now scrape the metrics of Fluentd on clusters that only have IPv6 addresses.
    - Fluent Bit can now listen on IPv6 addresses.

- The [OpenSearch Fluentd output]({{< relref "/docs/configuration/plugins/outputs/opensearch.md" >}}) now supports the `remove_keys` option.
- You can now set the `strategy` and `topologySpreadConstraints` in the Logging operator chart.

## Version 4.10

The following are the highlights and main changes of Logging operator 4.10. For a complete list of changes and bugfixes, see the [Logging operator 4.10 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.10.0)<!-- and the [Logging operator 4.9 release blog post](https://axoflow.com/logging-operator-4.9-release)-->.

- You can now control the memory usage of Fluent Bit in persistent buffering mode using the [`storage.max_chunks_up`]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md#bufferstorage-storage.max_chunks_up" >}}) option.

- When using [`systemdFilters`]({{< relref "/docs/configuration/crds/extensions/hosttailer_types.md#systemdtailer-systemdfilter" >}}) in `HostTailers`, you can now specify the journal field to use.

- The documentation of the [Gelf Fluentd output]({{< relref "/docs/configuration/plugins/outputs/gelf.md" >}}) has been improved, and now includes the `max_bytes` option that can limit the size of the messages.

- You can now configure image repository overrides in the syslog-ng spec (both in the `Logging` resource and in the `SyslogNGConfig` resource):

    ```yaml
    syslogNGImage:
      repository: ...
      tag: ...
    configReloadImage:
      repository: ...
      tag: ...
    metricsExporterImage:
      repository: ...
      tag: ...
    bufferVolumeMetricsImage:
      repository: ...
      tag: ...
    ```

- When using the [Kafka Fluentd output]({{< relref "/docs/configuration/plugins/outputs/kafka.md#kafka-max_send_limit_bytes" >}}), you can now set the maximal size of the messages using the `max_send_limit_bytes` option.

## Version 4.9

The following are the highlights and main changes of Logging operator 4.9. For a complete list of changes and bugfixes, see the [Logging operator 4.9 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.9.0)<!-- and the [Logging operator 4.9 release blog post](https://axoflow.com/logging-operator-4.9-release)-->.

### OpenTelemetry output

When using the [syslog-ng aggretor]({{< relref "/docs/configuration/output/_index.md#syslogngoutput" >}}), you can now send data directly to an OpenTelemetry endpoint. All metadata and the original log record are available in the body of the log record. Resource attributes will be available in a future release, when we switch to an OpenTelemetry input and receive standard OTLP logs. For details, see {{% xref "/docs/configuration/plugins/syslog-ng-outputs/opentelemetry.md" %}}.

```yaml
2024-07-05T09:00:23.407Z	info	LogsExporter	{"kind": "exporter", "data_type": "logs", "name": "debug", "resource logs": 1, "log records": 1}
2024-07-05T09:00:23.407Z	info	ResourceLog #0
Resource SchemaURL:
ScopeLogs #0
ScopeLogs SchemaURL:
InstrumentationScope
LogRecord #0
ObservedTimestamp: 2024-07-05 09:00:23.405798 +0000 UTC
Timestamp: 2024-07-05 09:00:23.406049 +0000 UTC
SeverityText:
SeverityNumber: Info2(10)
Body: Str({"ts":"2024-07-05T09:00:22.424670Z","log":"107.147.239.123 - - [05/Jul/2024:09:00:22 +0000] \"POST /index.html HTTP/1.1\" 200 14184 \"-\" \"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3191.0 Safari/537.36\" \"-\"\n","stream":"stdout","time":"2024-07-05T09:00:22.424670292Z","kubernetes":{"pod_name":"log-generator-55867b6d4c-66fdv","namespace_name":"log-generator","pod_id":"682c9ed9-9421-406f-9c7b-cf2b2e62f406","labels":{"app.kubernetes.io/instance":"log-generator","app.kubernetes.io/name":"log-generator","pod-template-hash":"55867b6d4c"},"host":"logging","container_name":"log-generator","docker_id":"dba14a358990c4b6ab82acdf75069952f3b180b3f16dd9527adc7eb11f6d2167","container_hash":"ghcr.io/kube-logging/log-generator@sha256:e26102ef2d28201240fa6825e39efdf90dec0da9fa6b5aea6cf9113c0d3e93aa","container_image":"ghcr.io/kube-logging/log-generator:0.7.0"}})
Trace ID:
Span ID:
Flags: 0
	{"kind": "exporter", "data_type": "logs", "name": "debug"}
```

### Elasticsearch data streams

You can now send messages and metrics to [Elasticsearch data streams](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-streams.html) to store your log and metrics data as time series data. You have to use the [syslog-ng aggretor]({{< relref "/docs/configuration/output/_index.md#syslogngoutput" >}}) to use this output. For details, see {{% xref "/docs/configuration/plugins/syslog-ng-outputs/elasticsearch_datastream.md" %}}.

### Improved Kafka performance

The {{% xref "/docs/configuration/plugins/outputs/kafka.md" %}} Fluentd output now supports using the rdkafka2 client, which offers higher performance than ruby-kafka. Set `use_rdkafka` to `true` to use the rdkafka2 client. (If you're using a custom Fluentd image, note that rdkafka2 requires v1.16-4.9-full or higher.)

### Containerd compatibility

As many users prefer Containerd instead of Docker as their container runtime, the slight differences between these CRIs are causing some problems. Now you can enable a compatibility layer with the `enableDockerParserCompatibilityForCRI` option of the `logging` CRD, for example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: containerd
spec:
  enableDockerParserCompatibilityForCRI: true
```

This option enables a log parser that is compatible with the docker parser. This has the following benefits:

- automatically parses JSON logs using the Merge_Log feature
- downstream parsers can use the `log` field instead of the `message` field, just like with the docker runtime
- the `concat` and `parser` filters are automatically set back to use the `log` field.

Here is a sample log message with the option enabled:

```json
{
  "log": "2024-08-12T14:19:29.672991171Z stderr F [2024/08/12 14:19:29] [ info] [input:tail:tail.0] inotify_fs_add(): inode=2939617 watch_fd=17 name=/var/log/containers/containerd-fluentd-0_default_fluentd-e46f1fcb1b63e7458fc43c079b7455f8e1305e551939ca128361a9574a194ed7.log",
  "kubernetes": {
    "pod_name": "containerd-fluentbit-mchwz",
    "namespace_name": "default",
    "pod_id": "22f86078-26f1-4202-bbc7-1dd5ddce20ec",
    "labels": {
      "app.kubernetes.io/instance": "containerd",
      "app.kubernetes.io/managed-by": "containerd",
      "app.kubernetes.io/name": "fluentbit",
      "controller-revision-hash": "6bb6f7f5f4",
      "pod-template-generation": "2"
    },
    "annotations": {
      "checksum/cri-log-parser.conf": "d902a0ee964e9398e637b581be851cdf50ab2846e82003d2f5e2feef82bef95d",
      "checksum/fluent-bit.conf": "dc9727d915c447b414dc05df2d9a6f23246cdca345309eb3107cc16ae8369b53"
    },
    "host": "logging",
    "container_name": "fluent-bit",
    "docker_id": "5cf032406344fdf41d76ce4489ee6f3ca092e9207ec49ab6209cc2bcf950e593",
    "container_image": "fluent/fluent-bit:3.0.4"
  }
}
```

### Set the severity of PrometheusRules

You can now configure `PrometheusRulesOverride` in your [`logging` CRDs]({{< relref "/docs/configuration/crds/v1beta1/common_types.md#PrometheusRulesOverride" >}}). The content of `PrometheusRulesOverride` is identical to the v1.Rule Prometheus rule type. The controller will match overrides by their names with the original rules. All of the override attributes are optional and whenever an attribute is set, it will replace the original attribute.

For example, you can change the severity of a rule like this:

```yaml
fluentd:
  metrics:
    prometheusRules: true
    prometheusRulesOverride:
    - alert: FluentdPredictedBufferGrowth
      labels:
         rulegroup: fluentd
         service:   fluentd
         severity:  none
```

### Other changes

- [Fluent Bit hot reload](#fluent-bit-hot-reload) now reloads imagePullSecrets as well.
- From now on, the entire `spec.Security.SecurityContext` is passed to Fluent Bit.
- Kubernetes namespace labels are added to the metadata by default. (The default of the `namespace_labels` option in the FluentBitAgent CRD is `on`.)

## Version 4.8

The following are the highlights and main changes of Logging operator 4.8. For a complete list of changes and bugfixes, see the [Logging operator 4.8 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.8.0) and the [Logging operator 4.8 release blog post](https://axoflow.com/logging-operator-4.8-release).

### Routing based on namespace labels

In your Fluentd ClusterFlows you can now route your messages based on namespace labels.

> Note: This feature requires a new Fluentd image: `ghcr.io/kube-logging/fluentd:v1.16-4.8-full`. If you're using a custom Fluentd image, make sure to update it!

If you have [enabled namespace labeling in Fluent Bit](https://kube-logging.dev/docs/whats-new/#kubernetes-namespace-labels-and-annotations), you can use namespace labels in your selectors, for example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: ClusterFlow
metadata:
  name: dev-logs
spec:
  match:
    - select:
        namespace_labels:
          tenant: dev
  globalOutputRefs:
    - example
```

### Breaking change

If you're using `hostTailer` or `eventTailer` and configured it through the helm chart's `logging.hostTailer` or `logging.eventTailer` option, note that now both components have an `enabled` flag. Set this flag to true if you used any of these components from the chart. For details, see the [pull request](https://github.com/kube-logging/logging-operator/pull/1576).

### Go templates in metrics-probe label values

You can now use go templates that resolve to destination information (`name`, `namespace`, `scope:local/global` and the `logging` name) in metrics-probe label values. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGClusterFlow
metadata:
  name: all
spec:
  match: {}
  outputMetrics:
    - key: custom_output
      labels:
        flow: all
        # define your own label for output name
        my-key-for-the-output: "{{ .Destination.Name }}"
        # do not add the output_name label to the metric
        output_name: ""
  globalOutputRefs:
    - http
```

### Other changes

- You can set the maximal number of TCP connections Fluent Bit can open towards the aggregator to avoid overloading it.

    ```yaml
    spec:
      controlNamespace: default
      fluentbit:
    # The below network configurations allow fluentbit to retry indefinitely on a limited number of connections to avoid overloading the aggregator (syslog-ng in this case)
      network:
        maxWorkerConnections: 2
      syslogng_output:
        Workers: 2
        Retry_Limit: "no_limits"
    ```

- In the Logging operator helm chart you can include extra manifests to deploy together with the chart using the `extraManifests` field, similarly to other popular charts.

## Version 4.7

The following are the highlights and main changes of Logging operator 4.7. For a complete list of changes and bugfixes, see the [Logging operator 4.7 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.7.0) and the [Logging operator 4.7 release blog post](https://axoflow.com/logging-operator-4.7-release).

### Breaking change for Fluentd

When using the Fluentd aggregator, Logging operator has overridden the default `chunk_limit_size` for the Fluentd disk buffers. Since Fluentd updated the default value to a much saner default, Logging operator won't override that to avoid creating too many small buffer chunks. (Having too many small chunks can lead to `too many open files` errors.)

This isn't an intrusive breaking change, it only affects your deployments if you intentionally or accidentally depended on this value.

### JSON output format for Fluentd

In addition to the default text format, Fluentd can now format the output as JSON:

```yaml
spec:
  fluentd:
    logFormat: json
```
<!-- FIXME why is this good / when is it needed? -->

### Disk buffer support for more outputs

Enabling disk buffers wasn't available for some of the outputs, this has been fixed for: [Gelf]({{< relref "/docs/configuration/plugins/outputs/gelf.md" >}}), [Elasticsearch]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/elasticsearch.md" >}}), [OpenObserve]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/openobserve.md" >}}), [S3]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/s3.md" >}}), [Splunk HEC]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/splunk_hec.md" >}}).

### Compression support for Elasticsearch

The [Elasticsearch output of the Fluentd aggregator]({{< relref "/docs/configuration/plugins/outputs/elasticsearch.md#elasticsearch-compression_level" >}}) now supports compressing the output data using gzip. You can use the `compression_level` option to set `default_compression`, `best_compression`, or `best_speed`. By default, compression is disabled.

### Protected ClusterOutputs for Fluentd

By default, ClusterOutputs can be referenced in any Flow. In certain scenarios, this means that users can send logs from Flows to the ClusterOutput, possibly spamming the output with user logs. From now on, you can set the `protected` flag for ClusterOutputs and prevent Flows from sending logs to the protected ClusterOutput.

### ConfigCheck settings for aggregators

You can now specify `configCheck` settings globally in the Loggings CRD, and override them if needed on the aggregator level in the [Fluentd]({{< relref "/docs/configuration/crds/v1beta1/fluentd_types.md" >}}) or [SyslogNG]({{< relref "/docs/configuration/crds/v1beta1/syslogng_types.md" >}}) CRD.

### Limit connections for Fluent Bit

You can now limit the number of TCP connections that each Fluent Bit worker can open toward the aggregator endpoints. The `max_worker_connections` is set to unlimited by default, and should be used together with the `Workers` option (which defaults to 2 according to the [Fluent Bit documentation](https://docs.fluentbit.io/manual/pipeline/outputs/tcp-and-tls#:~:text=double-,Workers,-Enables%20dedicated%20thread)). The following example uses a single worker with a single connection:

```yaml
kind: FluentbitAgent
spec:
  network:
    maxWorkerConnections: 1
  syslogng_output:
    Workers: 1
```

## Version 4.6

The following are the highlights and main changes of Logging operator 4.6. For a complete list of changes and bugfixes, see the [Logging operator 4.6 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.6.0) and the [Logging operator 4.6 release blog post](https://axoflow.com/fluent-bit-hot-reload-kubernetes-namespace-labels-vmware-outputs-logging-operator-4-6).

### Fluent Bit hot reload

As a Fluent Bit restart can take a long time when there are many files to index, Logging operator now supports [hot reload for Fluent Bit](https://docs.fluentbit.io/manual/administration/hot-reload) to reload its configuration on the fly.

You can enable hot reloads under the Logging's `spec.fluentbit.configHotReload` (legacy method) option, or the new FluentbitAgent's `spec.configHotReload` option:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: reload-example
spec:
  configHotReload: {}
```

You can configure the `resources` and `image` options:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: reload-example
spec:
  configHotReload:
    resources: ...
    image:
      repository: ghcr.io/kube-logging/config-reloader
      tag: v0.0.5
```

Many thanks to @aslafy-z for contributing this feature!

### VMware Aria Operations output for Fluentd

When using the Fluentd aggregator with the Logging operator, you can now send your logs to [VMware Aria Operations for Logs](https://www.vmware.com/products/aria-operations-for-logs.html). This output uses the [vmwareLogInsight plugin](https://github.com/vmware/fluent-plugin-vmware-loginsight).

Here is a sample output snippet:

```yaml
spec:
  vmwareLogInsight:
    scheme: https
    ssl_verify: true
    host: MY_LOGINSIGHT_HOST
    port: 9543
    agent_id: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
    log_text_keys:
	- log
	- msg
	- message
    http_conn_debug: false
```

Many thanks to @logikone for contributing this feature!

### VMware Log Intelligence output for Fluentd

When using the Fluentd aggregator with the Logging operator, you can now send your logs to [VMware Log Intelligence](https://aria.vmware.com/t/vmware-log-intelligence/). This output uses the [vmware_log_intelligence plugin](https://github.com/vmware/fluent-plugin-vmware-log-intelligence).

Here is a sample output snippet:

```yaml
spec:
  vmwarelogintelligence:
    endpoint_url: https://data.upgrade.symphony-dev.com/le-mans/v1/streams/ingestion-pipeline-stream
    verify_ssl: true
    http_compress: false
    headers:
      content_type: "application/json"
      authorization:
        valueFrom:
          secretKeyRef:
            name: vmware-log-intelligence-token
            key: authorization
      structure: simple
    buffer:
      chunk_limit_records: 300
      flush_interval: 3s
      retry_max_times: 3
```

Many thanks to @zrobisho for contributing this feature!

### Kubernetes namespace labels and annotations

Logging operator 4.6 supports the new Fluent Bit Kubernetes filter options that will be released in Fluent Bit 3.0. That way you'll be able to enrich your logs with Kubernetes namespace labels and annotations right at the source of the log messages.

Fluent Bit 3.0 hasn't been released yet (at the time of this writing), but you can use a developer image to test the feature, using a `FluentbitAgent` resource like this:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: namespace-label-test
spec:
  filterKubernetes:
    namespace_annotations: "On"
    namespace_labels: "On"
  image:
    repository: ghcr.io/fluent/fluent-bit
    tag: 3.0.0
```

### Other changes

- Enabling ServiceMonitor checks if Prometheus is already available.
- You can now use a custom PVC without a template for the statefulset.
- You can now configure PodDisruptionBudget for Fluentd.
- Event tailer metrics are now automatically exposed.
- You can configure [timeout-based configuration checks](https://kube-logging.dev/docs/whats-new/#timeout-based-configuration-checks) using the `logging.configCheck` object of the `logging-operator` chart.
- You can now specify the event tailer image to use in the `logging-operator` chart.
- Fluent Bit can now automatically delete irrecoverable chunks.
- The Fluentd statefulset and its components created by the Logging operator now include the whole securityContext object.
- The Elasticsearch output of the syslog-ng aggregator now supports the template option.
- To avoid problems that might occur when a tenant has a faulty output and backpressure kicks in, Logging operator now creates a dedicated tail input for each tenant.

### Removed feature

We have removed support for [Pod Security Policies (PSPs)](https://kubernetes.io/docs/concepts/security/pod-security-policy/), which were deprecated in Kubernetes v1.21, and removed from Kubernetes in v1.25.

Note that the API was left intact, it just doesn't do anything.

## Version 4.5

The following are the highlights and main changes of Logging operator 4.5. For a complete list of changes and bugfixes, see the [Logging operator 4.5 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.5.0).

### Standalone FluentdConfig and SyslogNGConfig CRDs

Starting with Logging operator version 4.5, you can either configure Fluentd in the `Logging` CR, or you can use a standalone `FluentdConfig` CR. Similarly, you can use a standalone `SyslogNGConfig` CRD to configure syslog-ng.

These standalone CRDs are namespaced resources that allow you to configure the Fluentd/syslog-ng aggregator in the control namespace, separately from the Logging resource. That way you can use a multi-tenant model, where tenant owners are responsible for operating their own aggregator, while the Logging resource is in control of the central operations team.

For details, see {{% xref "/docs/logging-infrastructure/fluentd.md" %}} and {{% xref "/docs/logging-infrastructure/syslog-ng.md" %}}.

### New syslog-ng features

When using syslog-ng as the log aggregator, you can now:

- Send data to [OpenObserve]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/openobserve.md" >}})
- Use a [custom date-parser]({{< relref "/docs/examples/date-parser.md" >}})
- Create [custom log metrics for sources and outputs]({{< relref "/docs/examples/custom-syslog-ng-metrics.md" >}})
- Set the permitted [SSL versions in HTTP based outputs]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/tls.md#tls-ssl_version" >}})
- Configure the [maxConnections parameter of the sources]({{< relref "/docs/configuration/crds/v1beta1/syslogng_types.md#syslogngspec-maxconnections" >}})

### New Fluentd features

When using Fluentd as the log aggregator, you can now:

- Use the [useragent Fluent filter]({{< relref "/docs/configuration/plugins/filters/useragent.md" >}})
- Configure [sidecar container in Fluentd pods]({{< relref "/docs/configuration/crds/v1beta1/fluentd_types.md#fluentdspec-sidecarcontainers" >}})
- Configure the [security-context of every container]({{< relref "/docs/configuration/crds/v1beta1/fluentd_types.md#fluentdspec-sidecarcontainers#fluentddrainconfig-securitycontext" >}})
- Set which [Azure Cloud to use]({{< relref "/docs/configuration/plugins/outputs/azurestore.md#output-config-azure_cloud" >}}) (for example, AzurePublicCloud), when using the Azure Storage output
- Customize the `image` to use in [event and host tailers]({{< relref "/docs/configuration/crds/extensions/_index.md" >}})

### Other changes

- LoggingStatus now includes the number (problemsCount) and the related watchNamespaces to help troubleshooting

### Image and dependency updates

For the list of images used in Logging operator, see {{% xref "/docs/image-versions.md" %}}.

## Version 4.4

The following are the highlights and main changes of Logging operator 4.4. For a complete list of changes and bugfixes, see the [Logging operator 4.4 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.4.0).

### New syslog-ng features

When using syslog-ng as the log aggregator, you can now use the following new outputs:

- [ElasticSearch]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/elasticsearch.md" >}})
- [Grafana Loki]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/loki.md" >}})
- [MongoDB]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/mongodb.md" >}})
- [Redis]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/redis.md" >}})
- [Amazon S3]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/s3.md" >}})
- [Splunk HEC]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/splunk_hec.md" >}})
- The [HTTP]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/http.md" >}}) output now supports the `log-fifo-size`, `response-action`, and `timeout` fields.

You can now use the `metrics-probe()` parser of syslog-ng in syslogNGFLow and SyslogNGClusterFlow. For details, see {{% xref "/docs/configuration/plugins/syslog-ng-filters/parser.md#metricsprobe" %}}.

### Multi-tenancy with namespace-based routing

Logging operator now supports namespace based routing for efficient aggregator-level multi-tenancy.

To learn more:

- Read the [overview about multi-tenancy]({{< relref "/docs/configuration/multitenancy/_index.md" >}}).
- Read the [details of the LoggingRoute]({{< relref "/docs/configuration/loggingroute.md" >}}) resource that enables this new behavior.
- Find a [simple example](https://github.com/kube-logging/logging-operator/tree/master/config/samples/multitenant-routing) to demonstrate the new behavior.

On a side note, nodegroup level isolation for hard multi-tenancy is also supported, see the {{% xref "docs/examples/multitenancy.md" %}} example.

### Forwarder logs

Fluent-bit now doesn't process the logs of the Fluentd and syslog-ng forwarders by default to avoid infinitely growing message loops. With this change, you can access Fluentd and syslog-ng logs simply by running `kubectl logs <name-of-forwarder-pod>`

In a future Logging operator version the logs of the aggregators will also be available for routing to external outputs.

### Timeout-based configuration checks

Timeout-based configuration checks are different from the normal method: they start a Fluentd or syslog-ng instance
without the dry-run or syntax-check flags, so output plugins or destination drivers actually try to establish
connections and will fail if there are any issues , for example, with the credentials.

Add the following to you `Logging` resource spec:

```yaml
spec:
  configCheck:
    strategy: StartWithTimeout
    timeoutSeconds: 5
```

### Istio support

For jobs/individual pods that run to completion, Istio sidecar injection needs to be disabled, otherwise the affected pods would live forever with the running sidecar container. Configuration checkers and Fluentd drainer pods can be configured with the label `sidecar.istio.io/inject` set to `false`. You can configure Fluentd drainer labels in the Logging spec.

### Improved buffer metrics

The buffer metrics are now available for both the Fluentd and the SyslogNG based aggregators.

The sidecar configuration has been rewritten to add a new metric and improve performance by avoiding unnecessary cardinality.

The name of the metric has been changed as well, but the original metric was kept in place to avoid breaking existing clients.

**Metrics currently supported by the sidecar**

Old
```
+# HELP node_buffer_size_bytes Disk space used [deprecated]
+# TYPE node_buffer_size_bytes gauge
+node_buffer_size_bytes{entity="/buffers"} 32253
```

New
```
+# HELP logging_buffer_files File count
+# TYPE logging_buffer_files gauge
+logging_buffer_files{entity="/buffers",host="all-to-file-fluentd-0"} 2
+# HELP logging_buffer_size_bytes Disk space used
+# TYPE logging_buffer_size_bytes gauge
+logging_buffer_size_bytes{entity="/buffers",host="all-to-file-fluentd-0"} 32253
```

### Other improvements

- You can now configure the resources of the buffer metrics sidecar.
- You can now rerun failed configuration checks if there is no configcheck pod.
- The [Fluentd ElasticSearch output]({{< relref "/docs/configuration/plugins/outputs/elasticsearch.md" >}}) now supports the [composable index template](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/index-templates.html) format. To use it, set the `use_legacy_template` option to `false`.
- The metrics for the syslog-ng forwarder are now exported using [axosyslog-metrics-exporter](https://github.com/axoflow/axosyslog-metrics-exporter).

### Image and dependency updates

For the list of images used in Logging operator, see {{% xref "/docs/image-versions.md" %}}.

Fluentd images with versions `v1.14` and `v1.15` are now EOL due to the fact they are based on ruby 2.7 which is EOL as well.

The currently supported image is [v1.15-ruby3](https://github.com/kube-logging/fluentd-images/tree/main/v1.15-ruby3) and build configuration for [v1.15-staging](https://github.com/kube-logging/fluentd-images/tree/main/v1.15-staging) is available for staging experimental changes.
