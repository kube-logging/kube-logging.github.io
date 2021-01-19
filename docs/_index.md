---
title: Logging operator
weight: 400
cascade:
  module: logging-operator
  githubEditUrl: "https://github.com/banzaicloud/logging-operator-docs/edit/master/docs/"
---

Welcome to the Logging operator documentation! The Logging operator is a core part of the [Banzai Cloud One Eye](https://banzaicloud.com/products/one-eye/) observability tool for Kubernetes.

## Overview

The Logging operator automates the deployment and configuration of a Kubernetes logging pipeline. The operator deploys and configures a Fluent Bit DaemonSet on every node to collect container and application logs from the node file system. Fluent Bit queries the Kubernetes API and enriches the logs with metadata about the pods, and transfers both the logs and the metadata to Fluentd. Fluentd receives, filters, and transfer logs to multiple outputs. Your logs will always be transferred on authenticated and encrypted channels.

This operator helps you bundle logging information with your applications: you can describe the behavior of your application in its charts, the Logging operator does the rest.

<p align="center"><img src="img/logging_operator_flow.png" ></p>

## Feature highlights

- Namespace isolation
- Native Kubernetes label selectors
- Secure communication (TLS)
- Configuration validation
- Multiple flow support (multiply logs for different transformations)
- Multiple [output]({{< relref "docs/one-eye/logging-operator/configuration/plugins/outputs/_index.md" >}}) support (store the same logs in multiple storage: S3, GCS, ES, Loki and more...)
- Multiple logging system support (multiple Fluentd, Fluent Bit deployment on the same cluster)

## Architecture

{{< include-headless "component-overview.md" "one-eye/logging-operator" >}}

For the detailed CRD documentation, see [List of CRDs]({{< relref "docs/one-eye/logging-operator/configuration/crds/_index.md" >}}).

![Logging operator architecture](/docs/one-eye/logging-operator/img/logging-operator-v2-architecture.png)

## Quickstart
<script id="asciicast-315998" src="https://asciinema.org/a/315998.js" async></script>

See also our [Quickstart guides](/docs/one-eye/logging-operator/quickstarts/).

## Support

The Logging operator is a core part of the [Banzai Cloud One Eye](https://banzaicloud.com/products/one-eye/) observability tool for Kubernetes. While the Logging operator itself is an [open-source project](https://github.com/banzaicloud/logging-operator/), the [Banzai Cloud One Eye](https://banzaicloud.com/products/one-eye/) product extends the functionality of the Logging operator with commercial features (for example, collecting host logs and Kubernetes events).

### Community support

If you encounter problems while using the Logging operator the documentation does not address, [open an issue](https://github.com/banzaicloud/logging-operator/issues) or talk to us on the Banzai Cloud Slack channel [#logging-operator](https://pages.banzaicloud.com/invite-slack).

### Commercial support

If you are using the Logging operator in a production environment and [require commercial support, contact Banzai Cloud](https://banzaicloud.com/contact/), the company backing the development of the Logging operator.
