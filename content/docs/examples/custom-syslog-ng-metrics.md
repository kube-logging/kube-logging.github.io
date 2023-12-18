---
title: Custom source and output metrics
linktitle: Custom syslog-ng metrics
weight: 1000
---

When using syslog-ng as the log aggregator, you can create custom log metrics for sources and outputs, based on the [metrics-probe() parser](https://axoflow.com/docs/axosyslog-core/chapter-parsers/metrics-probe/).

Available in Logging operator version 4.5 and later.

## Source metrics

Custom source metrics are added to the messages after the JSON parsing is completed. The following example adds the key called `custom_input`:

```yaml
kind: Logging
apiVersion: logging.banzaicloud.io/v1beta1
metadata:
  name: logging
spec:
  controlNamespace: default
  fluentbit: {}
  syslogNG:
    metrics: {}
    sourceMetrics:
      - key: custom_input
        labels:
          test: my-label-value
```

This corresponds to the following syslog-ng configuration:

```shell
source "main_input" {
    channel {
        source {
            network(flags("no-parse") port(601) transport("tcp") max-connections(100) log-iw-size(10000));
        };
        parser {
            json-parser(prefix("json."));
            metrics-probe(key("custom_input") labels(
                "logging" => "logging"
                "test" => "my-label-value"
            ));
        };
    };
};
```

And results in the following metrics:

```shell
curl logging-syslog-ng-0:9577/metrics  | grep custom_
# TYPE syslogng_custom_input gauge
syslogng_custom_input{logging="logging"} 154
```

## Output metrics

Output metrics are added before the log reaches the destination, and is decorated with the output metadata like: `name`, `namespace`, and `scope`. `scope` stores whether the output is a local or global one. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: all1
spec:
  match: {}
  outputMetrics:
    - key: custom_output
      labels:
        flow: all1
  localOutputRefs:
    - http
  globalOutputRefs:
    - http2
```

This corresponds to the following syslog-ng configuration:

```shell
filter "flow_default_all1_ns_filter" {
    match("default" value("json.kubernetes.namespace_name") type("string"));
};
log {
    source("main_input");
    filter("flow_default_all1_ns_filter");
    log {
        parser {
            metrics-probe(key("custom_output") labels(
                "flow" => "all1"
                "logging" => "logging"
                "output_name" => "http2"
                "output_namespace" => "default"
                "output_scope" => "global"
            ));
        };
        destination("clusteroutput_default_http2");
    };
    log {
        parser {
            metrics-probe(key("custom_output") labels(
                "flow" => "all1"
                "logging" => "logging"
                "output_name" => "http"
                "output_namespace" => "default"
                "output_scope" => "local"
            ));
        };
        destination("output_default_http");
    };
};
```

And results in the following metrics:

```shell
curl logging-syslog-ng-0:9577/metrics  | grep custom_
# TYPE syslogng_custom_output gauge
syslogng_custom_output{flow="all1",logging="logging",output_name="http2",output_namespace="default",output_scope="global"} 42
syslogng_custom_output{flow="all1",logging="logging",output_name="http",output_namespace="default",output_scope="local"} 42
syslogng_custom_output{flow="all2",logging="logging",output_name="http2",output_namespace="default",output_scope="global"} 154
```
