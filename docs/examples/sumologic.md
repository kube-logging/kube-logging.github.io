---
title: Sumo Logic with Logging operator and Fluentd
shorttitle: Sumo Logic with Fluentd
weight: 300
aliases:
    - /docs/one-eye/logging-operator/quickstarts/sumologic/
---

This guide walks you through a simple Sumo Logic setup using the Logging Operator.
Sumo Logic has Prometheus and logging capabilities as well. Now we only focus on the logging part.

## Configuration

There are 3 crucial plugins needed for a proper Sumo Logic setup.

1. Kubernetes metadata enhancer
2. Sumo Logic filter
3. Sumo Logic output

Let's setup the logging first.

### GlobalFilters

The first thing we need to ensure is that the `EnhanceK8s` filter is present in the `globalFilters` section of the Logging spec.
This adds additional data to the log lines (like deployment and service names).

```bash
kubectl apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: one-eye
spec:
  controlNamespace: logging
  enableRecreateWorkloadOnImmutableFieldChange: true
  globalFilters:
  - enhanceK8s: {}
  fluentbit:
    bufferStorage:
      storage.backlog.mem_limit: 256KB
    inputTail:
      Mem_Buf_Limit: 256KB
      storage.type: filesystem
    metrics:
      serviceMonitor: true
      serviceMonitorConfig: {}
  fluentd:
    disablePvc: true
    metrics:
      serviceMonitor: true
      serviceMonitorConfig: {}
EOF
```

### ClusterFlow

Now we can create a ClusterFlow. Add the Sumo Logic filter to the `filters` section of the ClusterFlow spec.
It will use the Kubernetes metadata and moves them to a special field called `_sumo_metadata`.
All those moved fields will be sent as HTTP Header to the Sumo Logic endpoint.

> Note: As we are using Fluent Bit to enrich Kubernetes metadata, we need to specify the field names where this data is stored.

```bash
kubectl -n logging apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: ClusterFlow
metadata:
  name: sumologic
spec:
  filters:
    - sumologic:
        source_name: kubernetes
        log_format: fields
        tracing_namespace: namespace_name
        tracing_pod: pod_name
  match:
  - select: {}
  globalOutputRefs:
    - sumo
EOF
```

### ClusterOutput

Create a Sumo Logic output secret from the URL.

```bash
kubectl create secret generic logging-sumo -n logging --from-literal "sumoURL=https://endpoint1.collection.eu.sumologic.com/......"
```

Finally create the Sumo Logic output.

```bash
kubectl -n logging apply -f - <<"EOF"
apiVersion: logging.banzaicloud.io/v1beta1
kind: ClusterOutput
metadata:
  name: sumo
spec:
  sumologic:
    buffer:
      flush_interval: 10s
      flush_mode: interval
    endpoint:
      valueFrom:
        secretKeyRef:
          name:  logging-sumo
          key: sumoURL
    source_name: kubernetes
EOF
```
