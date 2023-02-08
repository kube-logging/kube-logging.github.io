---
title: Collect Fluentd errors
shorttitle: Fluentd errors
weight: 1100
---

This section describes how to collect Fluentd error messages (messages that are sent to the [@ERROR label](https://docs.fluentd.org/configuration/config-file#error-label) from another plugin in Fluentd).

> Note: It depends on the specific plugin implementation what messages are sent to the @ERROR label. For example, a parsing plugin that fails to parse a line could send that line to the @ERROR label.

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

1. Set the `errorOutputRef` in the Logging resource to your preferred ClusterOutput.

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: one-eye
    spec:
      controlNamespace: default
      enableRecreateWorkloadOnImmutableFieldChange: true
      errorOutputRef: error-file
      fluentbit:
        bufferStorage: {}
        bufferStorageVolume:
          hostPath:
            path: ""
        filterKubernetes: {}
    # rest of the resource is omitted
    ```

    You cannot apply filters for this specific error flow.

1. Apply the ClusterOutput and Logging to your cluster.
