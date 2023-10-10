---
title: What's new
weight: 50
---

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
