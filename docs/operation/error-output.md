---
title: Collect Fluentd errors
shorttitle: Fluentd errors
weight: 1100
---

This section describes how to collect Fluentd error messages (that is, messages that have the [@ERROR label](https://docs.fluentd.org/configuration/config-file#error-label) in Fluentd).

## Prerequisites

Configuring readiness probes requires Logging operator 3.14.0 or newer installed on the cluster.

## Configure error output

To collect the error messages of Fluentd, complete the following steps.

1. Create a ClusterOutput that receives logs from every logging flow where error happens. For example, create a file output. For details on creating outputs, see {{% xref "../configuration/output.md" %}}.

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: ClusterOutput
    metadata:
      name: error-file
      namespace: default
        spec:
          file:
            path: /tmp/error.log
    ```

1. Create a ClusterFlow resource, and reference the error output in its **spec.errorOutputRef** section. The resource should contain only a reference to the error output, for example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: ClusterFlow
    metadata:
      name: error-flow
      namespace: default
    spec:
      controlNamespace: default
      errorOutputRef: error-file
    ```

    You cannot use filters or transform the messages in this flow.

1. Apply the ClusterOutput and the ClusterFlow to your cluster.
