---
title: Kubernetes Host Tailer
weight: 200
---

Kubernetes host tailer allows you to tail logs like `kubelet`, `audit` logs, or the `systemd` journal from the nodes.

![Host-tailer](/assets/blog/logging-extensions/logging-extensions-host-tailer.png)

## Create host tailer

To tail logs from the node's host filesystem, define one or more file tailers in the `host-tailer` configuration.

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

## Create file tailer

When an application (mostly legacy programs) is not logging in a Kubernetes-native way, Logging operator cannot process its logs. (For example, an old application does not send its logs to `stdout`, but uses some log files instead.) `File-tailer` helps to solve this problem: It configures Fluent Bit to tail the given file(s), and sends the logs to the `stdout`, to implement Kubernetes-native logging.

![Host-tailer](/assets/blog/logging-extensions/logging-extensions-host-tailer2.png)

However, `file-tailer` cannot access the pod's local dir, so the logfiles need to be written on a mounted volume.

Let's assume the following code represents a legacy application that generates logs into the `/legacy-logs/date.log` file. While the legacy-logs directory is mounted, it's accessible from other pods by mounting the same volume.

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - image: busybox
    name: test
    volumeMounts:
    - mountPath: /legacy-logs
      name: test-volume
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /legacy-logs/date.log;
          sleep 1;
        done
  volumes:
  - name: test-volume
    hostPath:
      path: /legacy-logs
EOF
```

To tail the logs of the previous example application, you can use the following `host-tailer` custom resource:

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: HostTailer
metadata:
  name: file-hosttailer-sample
spec:
  fileTailers:
    - name: sample-logfile
      path: /legacy-logs/date.log
      disabled: false
EOF
```

Logging operator configure the environment and start a `file-tailer` pod. It's also able to deal with multi-node clusters, since is starts the `host-tailer` pod through a `daemonset`.

Check the created file tailer pod:

```bash
kubectl get pod
```

The output should be similar to:

```bash
NAME                                       READY   STATUS    RESTARTS   AGE
file-hosttailer-sample-host-tailer-5tqhv   1/1     Running   0          117s
test-pod                                   1/1     Running   0          5m40s
```

Checking the logs of the `file-tailer's` pod. You will see the logfile's content on `stdout`. This way Logging operator can process those logs as well.

```bash
kubectl logs file-hosttailer-sample-host-tailer-5tqhv
```

The logs of the sample application should be similar to:

```bash
Fluent Bit v1.9.5
* Copyright (C) 2015-2022 The Fluent Bit Authors
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2022/09/13 12:26:02] [ info] [fluent bit] version=1.9.5, commit=9ec43447b6, pid=1
[2022/09/13 12:26:02] [ info] [storage] version=1.2.0, type=memory-only, sync=normal, checksum=disabled, max_chunks_up=128
[2022/09/13 12:26:02] [ info] [cmetrics] version=0.3.4
[2022/09/13 12:26:02] [ info] [sp] stream processor started
[2022/09/13 12:26:02] [ info] [output:file:file.0] worker #0 started
[2022/09/13 12:26:02] [ info] [input:tail:tail.0] inotify_fs_add(): inode=418051 watch_fd=1 name=/legacy-logs/date.log
Tue Sep 13 12:22:51 UTC 2022
Tue Sep 13 12:22:52 UTC 2022
Tue Sep 13 12:22:53 UTC 2022
Tue Sep 13 12:22:54 UTC 2022
Tue Sep 13 12:22:55 UTC 2022
Tue Sep 13 12:22:56 UTC 2022
```

### File Tailer configuration options

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| name | string | Yes | - | Name for the tailer<br> |
| path | string | No | - | Path to the loggable file<br> |
| disabled | bool | No | - | Disable tailing the file<br> |
| containerOverrides | *types.ContainerBase | No | - | Override container fields for the given tailer<br> |

## Tail systemd journal

This is a special case of `file-tailer`, since it tails the `systemd` journal file specifically.

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: HostTailer
metadata:
  name: systemd-tailer-sample
spec:
  # list of Systemd tailers
  systemdTailers:
    - name: my-systemd-tailer
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

## Example: Configure logging Flow to route logs from a host tailer

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

## Example: Kubernetes host tailer with multiple tailers

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

## Set custom priority

Create your own custom priority class in Kubernetes. Set its value between 0 and 2000000000. Note that:

- 0 is the default priority
- To change the default priority, set the `globalDefault` key.
- 2000000000 and above are reserved for the Kubernetes system
- PriorityClass is a non-namespaced object.

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

Now you can use your private priority class name to start hosttailer/eventtailer, for example:

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

## Configuration options

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| fileTailers | []FileTailer | No | - | List of file tailers<br> |
| c | []SystemdTailer | No | - | List of systemd tailers<br> |
| enableRecreateWorkloadOnImmutableFieldChange | bool | No | - | EnableRecreateWorkloadOnImmutableFieldChange enables the operator to recreate the<br>fluentbit daemonset and the fluentd statefulset (and possibly other resource in the future)<br>in case there is a change in an immutable field<br>that otherwise couldn't be managed with a simple update.<br> |
| workloadMetaOverrides | *types.MetaBase | No | - | Override metadata of the created resources<br> |
| workloadOverrides | *types.PodSpecBase | No | - | Override podSpec fields for the given daemonset<br> |

## Advanced configuration overrides

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
