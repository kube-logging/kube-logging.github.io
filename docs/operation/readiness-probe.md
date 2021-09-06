---
title: Readiness probe
weight: 1000
---

This section describes how to configure readiness probes for your Fluentd pods. If you don't configure custom readiness probes, Logging operator uses the default probes.

## Prerequisites

Configuring readiness probes requires Logging operator 3.14.0 or newer installed on the cluster.

## Overview of default readiness probes

By default, Logging operator performs the following readiness checks:

- Number of buffer files is too high (higher than 5000)
- Fluentd buffers are over 90% full

The parameters of the readiness probes and pod failure is set by using the usual [Kubernetes probe configuration parameters](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes). Instead of the Kubernetes defaults, the Logging operator uses the following values for these parameters:

```
InitialDelaySeconds: 5
TimeoutSeconds: 3
PeriodSeconds: 30
SuccessThreshold: 3
FailureThreshold: 1
```

Currently, you cannot modify the default readiness probes, because they are generated from the source files. For the detailed list of readiness probes, see the [source code of the Logging operator](https://github.com/banzaicloud/logging-operator/blob/master/pkg/sdk/api/v1beta1/fluentd_types.go). However, you can customize their values in the Logging custom resource, for example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
  name: logging-demo
spec:
  controlNamespace: logging
  fluentd:
    readinessDefaultCheck:
      bufferFileNumber: true
      bufferFileNumberMax: 5000
      bufferFreeSpace: true
      bufferFreeSpaceThreshold: 90
      failureThreshold: 1
      initialDelaySeconds: 5
      periodSeconds: 30
      successThreshold: 3
      timeoutSeconds: 3
```

## Add custom readiness probes {#custom-readiness-probes}

You can add your own custom readiness probes to the **spec.ReadinessProbe** section of the **logging** custom resource. For details on the format of readiness probes, see the [official Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes).

Note that if you set custom readiness probes, they completely override the default probes.

The Logging operator applies the following readiness probe by default:

```yaml
 readinessProbe:
      exec:
        command:
        - /bin/sh
        - -c
        - FREESPACE_THRESHOLD=90
        - FREESPACE_CURRENT=$(df -h $BUFFER_PATH  | grep / | awk '{ print $5}' | sed
          's/%//g')
        - if [ "$FREESPACE_CURRENT" -gt "$FREESPACE_THRESHOLD" ] ; then exit 1; fi
        - MAX_FILE_NUMBER=5000
        - FILE_NUMBER_CURRENT=$(find $BUFFER_PATH -type f -name *.buffer | wc -l)
        - if [ "$FILE_NUMBER_CURRENT" -gt "$MAX_FILE_NUMBER" ] ; then exit 1; fi
      failureThreshold: 1
      initialDelaySeconds: 5
      periodSeconds: 30
      successThreshold: 3
      timeoutSeconds: 3
```
