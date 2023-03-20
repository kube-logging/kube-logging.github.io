---
title: Examples
weight: 330
---

## Flow examples

The following examples show some simple flows. For more examples that use filters, see {{% xref "/docs/examples/filters-in-flows.md" %}}.

### Flow with a single output

This Flow sends every message with the `app: nginx` label to the output called `forward-output-sample`.

{{< include-code "logging_flow_single_output.yaml" "yaml" >}}

### Flow with multiple outputs

This Flow sends every message with the `app: nginx` label to the `gcs-output-sample` nad `s3-output-example` outputs.

{{< include-code "logging_flow_multiple_output.yaml" "yaml" >}}

## Logging examples

Simple [Logging]({{< relref "/docs/logging-infrastructure/logging.md" >}}) definition with default values.

{{< include-code "logging_logging_simple.yaml" "yaml" >}}

### Logging with TLS

Simple [Logging]({{< relref "/docs/logging-infrastructure/logging.md" >}}) definition with [TLS encryption enabled]({{< relref "/docs/logging-infrastructure/tls.md" >}}).

{{< include-code "logging_logging_tls.yaml" "yaml" >}}

## Output examples

### Simple file output

Defines a [file output]({{< relref "/docs/configuration/plugins/outputs/file.md" >}}) with timestamped log files.

{{< include-code "logging_output_file.yaml" "yaml" >}}

### Drop messages into dev/null output

Creates a dev/null output that can be the destination of messages you want to drop explicitly.

{{< warning >}}Messages sent to this output are irrevocably lost forever.{{< /warning >}}

{{< include-code "logging_output_null.yaml" "yaml" >}}

### S3 output

Defines an [Amazon S3 output]({{< relref "/docs/configuration/plugins/outputs/s3.md" >}}) to store your logs in a bucket.

{{< include-code "logging_output_s3.yaml" "yaml" >}}

<!-- FIXME group and list other yamls, add descriptions to the examples, link reference docs -->

### GCS output

Defines a [Google Cloud Storage output]({{< relref "/docs/configuration/plugins/outputs/gcs.md" >}}) to store your logs.

{{< include-code "logging_output_s3.yaml" "yaml" >}}