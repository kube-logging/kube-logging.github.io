---
title: What's new
weight: 50
---

## Version 4.4

The following are the highlights and main changes of Logging operator 4.4. For a complete list of changes and bugfixes, see the [Logging operator 4.4 releases page](https://github.com/kube-logging/logging-operator/releases/tag/4.4.0).

### New syslog-ng features

When using syslog-ng as the log aggregator, you can now use the following new outputs:

- [ElasticSearch]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/elasticsearch.md" >}})
- [MongoDB]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/mongodb.md" >}})
- [Redis]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/redis.md" >}})
- [Splunk HEC]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/splunk_hec.md" >}})
- The [HTTP]({{< relref "/docs/configuration/plugins/syslog-ng-outputs/http.md" >}}) output now supports the `log-fifo-size`, `response-action`, and `timeout` fields.

You can now use the `metrics-probe()` parser of syslog-ng in syslogNGFLow and SyslogNGClusterFlow. For details, see {{% xref "/docs/configuration/plugins/syslog-ng-filters/parser.md#metricsprobe" %}}.

### Multitenancy and namespace-based routing

Logging operator now supports hard multitenancy and namespace-based routing. For an example configuration, see the [sample configuration files](https://github.com/kube-logging/logging-operator/tree/master/config/samples/mulitenant-hard).

### Forwarder logs

Logging operator now doesn't forward the logs of the Fluentd and syslog-ng forwarders to the configured outputs to avoid infinite message loops. The logs of Fluentd and syslog-ng are now sent to the standard output of their pods, so you can access them by running `kubectl logs <name-of-forwarder-pod>`

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

By default, Logging operator now doesn't inject the istio sidecar into jobs/individual pods that run to completion. Configuration checkers and Fluentd drainer pods now have `sidecar.istio.io/inject` set to `false` by default. You can configure Fluentd drainer labels in the Logging spec.

For non-istio users, these changes make no difference, as this label is only used in Istio context. For Istio users, these defaults make Logging operator work out of the box.

## Other improvements

- You can now configure the resources of the buffer metrics sidecar.
- You can now rerun failed configuration checks if there is no configcheck pod.
- The [Fluentd ElasticSearch output]({{< relref "/docs/configuration/plugins/outputs/elasticsearch.md" >}}) now supports the [composable index template](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/index-templates.html) format. To use it, set the `use_legacy_template` option to `false`.
- The metrics for the syslog-ng forwarder are now exported using [axosyslog-metrics-exporter](https://github.com/axoflow/axosyslog-metrics-exporter).

### Image and dependency updates

For the list of images used in Logging operator, see {{% xref "/docs/image-versions.md" %}}.
