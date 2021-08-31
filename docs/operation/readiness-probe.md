---
title: Readiness probe
weight: 1000
---

This section describes how to configure readiness probes for your Fluentd pods. If you don't configure custom readiness probes, Logging operator uses the default probes.

## Prerequisites

Configuring readiness probes requires Logging operator 3.14.0 or newer installed on the cluster.

## Overview of default readiness probes

By default, Logging operator performs the following readiness checks:

- Number of buffer files is too high
- Fluentd buffers are over 90% full

Currently, you cannot modify the default readiness probes, because they are generated from the source files. For the detailed list of readiness probes, see the [source code of the Logging operator](https://github.com/banzaicloud/logging-operator/blob/master/pkg/sdk/api/v1beta1/fluentd_types.go).

## Add custom readiness probes {#custom-readiness-probes}

You can add your own custom readiness probes to the **spec.ReadinessProbe** section of the **logging** custom resource. For details on the format of readiness probes, see the [official Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes).

Note that if you set custom readiness probes, they completely override the default probes.
