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

You can now use timeout-based configuration strategies for both syslog-ng and Fluentd. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: all-to-file
spec:
  configCheck:
    strategy: StartWithTimeout
    timeoutSeconds: 5
  syslogNG: {}
  controlNamespace: default
```

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: all-to-file
spec:
  configCheck:
    strategy: StartWithTimeout
    timeoutSeconds: 5
  fluentd: {}
  controlNamespace: default
```

### Istio support

By default, Logging operator adds a label to avoid injecting the Istio sidecar into jobs/individual pods that run to completion. Configuration checkers and Fluentd drainer pods now have `sidecar.istio.io/inject` set to `false` by default. You can configure Fluentd drainer labels in the Logging spec.

For non-istio users, these changes make no difference, as this label is only used in Istio context. For Istio users, these defaults make Logging operator work out of the box.

## Other improvements

- You can now configure the resources of the buffer metrics sidecar.
- You can now rerun failed configuration checks if there is no configcheck pod.
- The [Fluentd ElasticSearch output]({{< relref "/docs/configuration/plugins/outputs/elasticsearch.md" >}}) now supports the [composable index template](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/index-templates.html) format. To use it, set the `use_legacy_template` option to `false`.
- The metrics for the syslog-ng forwarder are now exported using [axosyslog-metrics-exporter](https://github.com/axoflow/axosyslog-metrics-exporter).

### Image and dependency updates

For the list of images used in Logging operator, see {{% xref "/docs/image-versions.md" %}}.

Fluentd images with versions `v1.14` and `v1.15` are now EOL due to the fact they are based on ruby 2.7 which is EOL as well.

The currently supported image is [v1.15-ruby3](https://github.com/kube-logging/fluentd-images/tree/main/v1.15-ruby3) and build configuration for [v1.15-staging](https://github.com/kube-logging/fluentd-images/tree/main/v1.15-staging) is available for staging experimental changes.
