# Logging Operator & Sumologic

What is the same and what is different of sumologic

Sumnologic add Prometheus and logging capabilities as well. We will only talk about the logging part not the metrics.


## Configuration

### GlobalFilters

The first thing we need to ensure is that the EnhanceK8s filter is added to the Logging spec globalFilters section.
This will ensure to add additional data to the log lines (like deployment and service names)

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

ClusterFlow

Now we can create a ClusterFlow. The sumologic filter will use the Kubernetes metadata and moves them in a special field.
All those moved fields will be sent as HTTP Header to the sumologic.

> Note: As we are using fluentbit Kubernetes metadata we need to specify the field names where the metada is stored.
>

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

1. Create a Sumologic output secret from the URL.

```bash
kubectl create secret generic logging-sumo -n logging --from-literal "sumoURL=https://endpoint1.collection.eu.sumologic.com/......"
```


ClusterOutput

Finally we need a sumologic output.
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
