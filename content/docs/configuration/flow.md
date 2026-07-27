---
title: Flow and ClusterFlow
weight: 100
---

Flows route the selected log messages to the specified outputs. Depending on which log forwarder you use, you can use different filters and outputs, and have to configure different custom resources.

## Fluentd flows

`Flow` defines a logging flow for **Fluentd** with **filters** and **outputs**.

The Flow is a `namespaced` resource, so only logs from the same namespaces are collected. You can specify `match` statements to select or exclude logs according to Kubernetes `labels`, container and host names. (Match statements are evaluated in the order they are defined and processed only until the first matching `select` or `exclude` rule applies.) For detailed examples on using the match statement, see [log routing]({{< relref "/docs/configuration/log-routing.md" >}}).

You can define one or more `filters` within a Flow. Filters can perform various actions on the logs, for example, add additional data, transform the logs, or parse values from the records.
The filters in the flow are applied in the order in the definition. You can find the [list of supported filters here]({{< relref "/docs/configuration/plugins/filters">}}).

### Raw Fluentd filter

Use the `raw` filter to inject native Fluentd filter configuration when the Logging operator doesn't expose the filter you need as a first-class plugin. It is an escape hatch for custom or otherwise unsupported Fluentd filters.

{{< warning >}}Raw configuration bypasses the operator's schema and validation. Use it only when no first-class filter fits your use case.{{< /warning >}}

The `raw` filter is gated by the `spec.enableRawFluentdFilter` field (a boolean that defaults to `false`) on the `Logging` resource, not by an operator CLI flag. If a `Flow` or `ClusterFlow` uses a `raw` filter while this field is `false`, config generation fails and the affected `Flow` gets an entry in its `status.problems`.

Put the configuration in `raw.config`, a string of native Fluentd filter configuration. The body must include an `@type` line and must not be wrapped in the enclosing `<filter>...</filter>` tags: the operator adds the `<filter **>` directive and sets (overwrites) the `@id`. The `raw.config` string is capped at 64 KiB and 32 levels of nesting.

You can use the `raw` filter in the `filters` list of a `Flow` or `ClusterFlow` (as well as in `globalFilters` and the default flow), like any other filter. For details, see the generated `raw` filter reference in the [list of supported filters]({{< relref "/docs/configuration/plugins/filters">}}).

First, enable raw Fluentd filters on the `Logging` resource:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
  namespace: logging
spec:
  enableRawFluentdFilter: true
  fluentd: {}
  fluentbit: {}
  controlNamespace: logging
```

The following example injects a custom Fluentd filter that the operator does not expose. The `@type anonymizer` filter is illustrative — the point is the shape of the configuration: an `@type` line, no enclosing `<filter>` tags, and nested directives where needed.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: raw-filter-sample
  namespace: default
spec:
  filters:
    - raw:
        config: |
          @type anonymizer
          @log_level info
          <mask ipaddr>
            keys client_ip
            type network
          </mask>
  match:
    - select:
        labels:
          app: nginx
  localOutputRefs:
    - s3-output
```

At the end of the Flow, you can attach one or more [outputs]({{< relref "/docs/configuration/output.md" >}}), which may also be `Output` or `ClusterOutput` resources.

> `Flow` resources are `namespaced`, the `selector` only select `Pod` logs within namespace.
> `ClusterFlow` defines a Flow **without** namespace restrictions. It is also only effective in the `controlNamespace`.
 `ClusterFlow` selects logs from **ALL** namespace.

The following example transforms the log messages from the `default` namespace and sends them to an S3 output.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: flow-sample
  namespace: default
spec:
  filters:
    - parser:
        remove_key_name_field: true
        parse:
          type: nginx
    - tag_normaliser:
        format: ${namespace_name}.${pod_name}.${container_name}
  localOutputRefs:
    - s3-output
  match:
    - select:
        labels:
          app: nginx
```

> Note: In a multi-cluster setup you cannot easily determine which cluster the logs come from. You can append your own labels to each log
using the [record modifier filter]({{< relref "/docs/configuration/plugins/filters/record_modifier.md" >}}).

- For the details of `Flow` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/flow_types.md" %}}.
- For the details of `ClusterFlow` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/clusterflow_types.md" %}}.
- For details on selecting messages, see {{% xref "/docs/configuration/log-routing.md" %}}
- See the [list of supported filters]({{< relref "/docs/configuration/plugins/filters">}}).

## syslog-ng flows {#syslogngflow}

`SyslogNGFlow` defines a logging flow for **syslog-ng** with **filters** and **outputs**.

{{< include-headless "syslog-ng-minimum-version.md" >}}

The Flow is a `namespaced` resource, so only logs from the same namespaces are collected. You can specify `match` statements to select or exclude logs according to Kubernetes `labels`, container and host names. For detailed examples on using the match statement, see [log routing with syslog-ng]({{< relref "/docs/configuration/log-routing-syslog-ng.md" >}}).

You can define one or more filters within a Flow. Filters can perform various actions on the logs, for example, add additional data, transform the logs, or parse values from the records.
The filters in the flow are applied in the order in the definition. You can find the [list of supported filters here]({{< relref "/docs/configuration/plugins/syslog-ng-filters/_index.md">}}).

At the end of the Flow, you can attach one or more [outputs]({{< relref "/docs/configuration/output.md" >}}), which may also be `Output` or `ClusterOutput` resources.

> `SyslogNGFlow` resources are `namespaced`, the `selector` only selects `Pod` logs within the namespace.
> `SyslogNGClusterFlow` defines a SyslogNGFlow **without** namespace restrictions. It is also only effective in the `controlNamespace`.
 `SyslogNGClusterFlow` selects logs from **ALL** namespaces.

The following example selects only messages sent by the log-generator application and forwards them to a syslog output.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: TestFlow
  namespace: default
spec:
  match:
    and:
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/instance
        pattern: log-generator
        type: string
    - regexp:
        value:  json.kubernetes.labels.app.kubernetes.io/name
        pattern: log-generator
        type: string
  localOutputRefs:
    - syslog-output
```

- For the details of the `SyslogNGFlow` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/syslogng_flow_types.md" %}}.
- For the details of the `SyslogNGClusterFlow` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/syslogng_clusterflow_types.md" %}}.
- For details on selecting messages, see {{% xref "/docs/configuration/log-routing-syslog-ng.md" %}}
- See the [list of supported filters]({{< relref "/docs/configuration/plugins/syslog-ng-filters/_index.md" >}}).
