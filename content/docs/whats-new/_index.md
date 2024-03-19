---
title: What's new
weight: 50
---

## Version 4.6

The following are the highlights and main changes of Logging operator 4.6. For a complete list of changes and bugfixes, see the [Logging operator 4.6 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.6.0) and the [Logging operator 4.6 release blog post](fluent-bit-hot-reload-kubernetes-namespace-labels-vmware-outputs-logging-operator-4-6).

## Fluent Bit hot reload

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

## VMware Aria Operations output for Fluentd

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

## VMware Log Intelligence output for Fluentd

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

## Kubernetes namespace labels and annotations

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
    repository: ghcr.io/fluent/fluent-bit/unstable
    tag: latest
```

## Other changes

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

## Removed feature

We have removed support for [Pod Security Policies (PSPs)](https://kubernetes.io/docs/concepts/security/pod-security-policy/), which were deprecated in Kubernetes v1.21, and removed from Kubernetes in v1.25.

Note that the API was left intact, it just doesn't do anything.

## Version 4.5

The following are the highlights and main changes of Logging operator 4.5. For a complete list of changes and bugfixes, see the [Logging operator 4.5 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.5.0).

## Standalone FluentdConfig and SyslogNGConfig CRDs

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

## Other changes

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

### Multitenancy with namespace-based routing

Logging operator now supports namespace based routing for efficient aggregator-level multi-tenancy.

In the project repository you can:
- find an [overview about multitenancy](https://github.com/kube-logging/logging-operator/blob/master/docs/multi-tenancy.md).
- find more detailed information about the new [LoggingRoute](https://github.com/kube-logging/logging-operator/blob/master/docs/logging-route.md) resource that enables this new behaviour.
- find a [simple example](https://github.com/kube-logging/logging-operator/tree/master/config/samples/multitenant-routing) to demonstrate the new behaviour

On a side note, nodegroup level isolation for hard multitenancy is also supported, see the {{% xref "docs/examples/multitenancy.md" %}} example.

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

## Other improvements

- You can now configure the resources of the buffer metrics sidecar.
- You can now rerun failed configuration checks if there is no configcheck pod.
- The [Fluentd ElasticSearch output]({{< relref "/docs/configuration/plugins/outputs/elasticsearch.md" >}}) now supports the [composable index template](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/index-templates.html) format. To use it, set the `use_legacy_template` option to `false`.
- The metrics for the syslog-ng forwarder are now exported using [axosyslog-metrics-exporter](https://github.com/axoflow/axosyslog-metrics-exporter).

### Image and dependency updates

For the list of images used in Logging operator, see {{% xref "/docs/image-versions.md" %}}.

Fluentd images with versions `v1.14` and `v1.15` are now EOL due to the fact they are based on ruby 2.7 which is EOL as well.

The currently supported image is [v1.15-ruby3](https://github.com/kube-logging/fluentd-images/tree/main/v1.15-ruby3) and build configuration for [v1.15-staging](https://github.com/kube-logging/fluentd-images/tree/main/v1.15-staging) is available for staging experimental changes.
