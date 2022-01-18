---
title: Kubernetes Event Tailer
weight: 100
---

Kubernetes events are objects that provide insight into what is happening
inside a cluster, such as what decisions were made by the scheduler or
why some pods were evicted from the node.

## Configuration options

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| controlNamespace | string | Yes | - | The resources of Eventtailer will be placed into this namespace<br> |
| positionVolume | volume.KubernetesVolume | No | - | Volume definition for tracking fluentbit file positions (optional)<br> |
| workloadMetaOverrides | *types.MetaBase | No | - | Override metadata of the created resources<br> |
| workloadOverrides | *types.PodSpecBase | No | - | Override podSpec fields for the given statefulset<br> |

## Examples

### Example: Configure Kubernetes event tailer

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: EventTailer
metadata:
  name: sample
spec:
  controlNamespace: default
EOF
```

### Example: Configure Kubernetes event tailer with PVC

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: EventTailer
metadata:
  name: sample
spec:
  controlNamespace: default
  positionVolume:
    pvc:
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
        volumeMode: Filesystem
EOF
```

### Example: Configure logging Flow to route logs from an event tailer

The following example configures a flow to route the previously created `sample-eventtailer` EventTailer's log. 

```bash
kubectl apply -f - <<EOF
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: eventtailer-flow
  namespace: default
spec:
  filters:
  - tag_normaliser: {}
  match:
  # keeps data matching to label, the rest of the data will be discarded by this flow implicitly
  - select:
      labels:
        app.kubernetes.io/name: sample-event-tailer
  outputRefs:
    - sample-output
EOF
```
