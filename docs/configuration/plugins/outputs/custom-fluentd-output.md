---
title: How to add a custom FluentD output plugin to Logging operator
shorttitle: Custom FluentD plugin
weight: 1000
---

If there is a FluentD plugin that the Logging operator does not support (for example, [https://github.com/cloudfoundry-community/fluent-plugin-nats](https://github.com/cloudfoundry-community/fluent-plugin-nats)), here is how you can make it work with the Logging operator. Note that this is not a detailed tutorial, just a high-level overview of the process.

1. Add the plugin to the latest FluentD dockerfile used by the Logging operator [for example, at the moment v1.11](https://github.com/banzaicloud/logging-operator/blob/master/fluentd-image/v1.11/Dockerfile).
1. Write the Go code to have the Output CRD support the new plugin. You can use the [Redis plugin pull request](https://github.com/banzaicloud/logging-operator/pull/549/files) as an example.
1. Generate the custom resource definitions and the CRD documentation from the Go code by running `make generate manifests`
    If you want to see what has changed, run `make check-diff`

For more details, see {{% xref "/docs/one-eye/logging-operator/developers.md" %}}.
