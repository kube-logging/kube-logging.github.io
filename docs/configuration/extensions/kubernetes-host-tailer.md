---
title: Kubernetes Host Tailer
weight: 200
aliases:
    - /docs/logging-extensions/kubernetes-host-tailer/
---

Kubernetes host tailer allows you to tail logs like `kubelet`, `audit` logs, or the `systemd` journal from the nodes.

## Kubernetes Systemd tailer

Tail logs from the systemd journal. Define one or more systemd tailers in the `Observer`
configuration.

### Example: configuration Systemd tailer

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: HostTailer
metadata:
  name: systemd-hosttailer-sample
spec:
  systemdTailers:
    - name: my-systemd-tailer
      disabled: false
      maxEntries: 100
      systemdFilter: kubelet.service
EOF
```

### Systemd tailer configuration options

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| name | string | Yes | - | Name for the tailer<br> |
| path | string | No | - | Override systemd log path<br> |
| disabled | bool | No | - | Disable component<br> |
| systemdFilter | string | No | - | Filter to select systemd unit example: kubelet.service<br> |
| maxEntries | int | No | - | Maximum entries to read when starting to tail logs to avoid high pressure<br> |
| containerOverrides | *types.ContainerBase | No | - | Override container fields for the given tailer<br> |

## Kubernetes Host File tailer {#host-file-tailer}

Tail logs from the node's host filesystem. Define one or more file tailers in the `HostTailer`
configuration.

### Example: Configure host File Tailer

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: HostTailer
metadata:
  name: file-hosttailer-sample
spec:
  fileTailers:
    - name: nginx-access
      path: /var/log/nginx/access.log
      disabled: false
EOF
```

### File Tailer configuration options

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| name | string | Yes | - | Name for the tailer<br> |
| path | string | No | - | Path to the loggable file<br> |
| disabled | bool | No | - | Disable tailing the file<br> |
| containerOverrides | *types.ContainerBase | No | - | Override container fields for the given tailer<br> |

### Example: Configure logging Flow to route logs from a Hosttailer

The following example uses the flow's match term to listen the previously created `file-hosttailer-sample` Hosttailer's log. 

```bash
kubectl apply -f - <<EOF
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: hosttailer-flow
  namespace: default
spec:
  filters:
  - tag_normaliser: {}
  # keeps data matching to label, the rest of the data will be discarded by this flow implicitly
  match:
  - select:
      labels: 
        app.kubernetes.io/name: file-hosttailer-sample
      # there might be a need to match on container name too (in case of multiple containers)
      container_names:
        - nginx-access
  outputRefs:
    - sample-output
EOF
```

### Example: Kubernetes host tailer with multiple tailers

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: HostTailer
metadata:
  name: multi-sample
spec:
  # list of File tailers
  fileTailers:
    - name: nginx-access
      path: /var/log/nginx/access.log
    - name: nginx-error
      path: /var/log/nginx/error.log
  # list of Systemd tailers
  systemdTailers:
    - name: my-systemd-tailer
      maxEntries: 100
      systemdFilter: kubelet.service
EOF
```

### Example: Setting up custom priority

Create your own custom priority class in Kubernetes. Set its value between 0 and 2000000000.
>Priority Hints:
>* 0 is the default priority
>* To change the default priority, set the `globalDefault` key
>* 2000000000 and above are reserved for kubernetes system
>* PriorityClass is a non-namespaced object

```bash
kubectl apply -f - <<EOF
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: hosttailer-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for hosttailer pods only."
EOF
```

Now you can use your private priority class name to start hosttailer/eventtailer as it shown below:

```bash
kubectl apply -f -<<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: HostTailer
metadata:
  name: priority-sample
spec:
  controlNamespace: default
  # Override podSpecBase variables here
  workloadOverrides:
    priorityClassName: hosttailer-priority
  fileTailers:
    - name: nginx-access
      path: /var/log/nginx/access.log
    - name: nginx-error
      path: /var/log/nginx/error.log
EOF
```

### Configuration options

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| fileTailers | []FileTailer | No | - | List of file tailers<br> |
| c | []SystemdTailer | No | - | List of systemd tailers<br> |
| enableRecreateWorkloadOnImmutableFieldChange | bool | No | - | EnableRecreateWorkloadOnImmutableFieldChange enables the operator to recreate the<br>fluentbit daemonset and the fluentd statefulset (and possibly other resource in the future)<br>in case there is a change in an immutable field<br>that otherwise couldn't be managed with a simple update.<br> |
| workloadMetaOverrides | *types.MetaBase | No | - | Override metadata of the created resources<br> |
| workloadOverrides | *types.PodSpecBase | No | - | Override podSpec fields for the given daemonset<br> |

### Advanced configuration overrides

### MetaBase

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| annotations | `map[string]string` | No | - |  |
| labels | `map[string]string` | No | - |  |

### PodSpecBase

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| tolerations | `[]corev1.Toleration` | No | - |  |
| nodeSelector | `map[string]string` | No | - |  |
| serviceAccountName | `string` | No | - |  |
| affinity | `*corev1.Affinity` | No | - |  |
| securityContext | `*corev1.PodSecurityContext` | No | - |  |
| volumes | `[]corev1.Volume` | No | - |  |
| priorityClassName | `string` | No | - |  |

### ContainerBase

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| resources | `*corev1.ResourceRequirements` | No | - |  |
| image | `string` | No | - |  |
| pullPolicy | `corev1.PullPolicy` | No | - |  |
| command | `[]string` | No | - |  |
| volumeMounts | `[]corev1.VolumeMount` | No | - |  |
| securityContext | `*corev1.SecurityContext` | No | - |  |
