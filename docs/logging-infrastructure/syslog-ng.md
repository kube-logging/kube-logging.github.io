---
title: Configure syslog-ng
shorttitle: syslog-ng log forwarder
weight: 300
---

{{< include-headless "syslog-ng-minimum-version.md" "one-eye/logging-operator" >}}

You can configure the deployment of the syslog-ng log forwarder via the **syslogNG** section of the {{% xref "/docs/one-eye/logging-operator/logging-infrastructure/logging.md" %}}. For the detailed list of available parameters, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/syslogng_types.md" %}}.

The following example sets a volume mount that syslog-ng can use for buffering messages on the disk (if {{% xref "/docs/one-eye/logging-operator/configuration/plugins/syslog-ng-outputs/disk_buffer.md" %}} is configured in the output).

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
  name: test
spec:
  syslogNG:
    statefulSet:
      spec:
        template:
          spec:
            containers:
            - name: syslog-ng
              volumeMounts:
              - mountPath: /buffers
                name: buffer
        volumeClaimTemplates:
        - metadata:
            name: buffer
          spec:
            accessModes:
            - ReadWriteOnce
            resources:
              requests:
                storage: 10Gi
```

{{< include-headless "cpu-memory-requirements.md" "one-eye/logging-operator" >}}

## Probe

A [Probe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) is a diagnostic performed periodically by the kubelet on a Container. To perform a diagnostic, the kubelet calls a Handler implemented by the Container. You can configure a probe for syslog-ng in the **livenessProbe** section of the {{% xref "/docs/one-eye/logging-operator/logging-infrastructure/logging.md" %}}. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  syslogNG:
    livenessProbe:
      periodSeconds: 60
      initialDelaySeconds: 600
      exec:
        command:
        - "/usr/sbin/syslog-ng-ctl"
        - "--control=/tmp/syslog-ng/syslog-ng.ctl"
        - "query"
        - "get"
        - "global.sdata_updates.processed"
  controlNamespace: logging
```

You can use the following parameters:

| Name                    | Type           | Default | Description |
|-------------------------|----------------|---------|-------------|
| initialDelaySeconds | int | 30 | Number of seconds after the container has started before liveness probes are initiated. |
| timeoutSeconds | int | 0 | Number of seconds after which the probe times out. |
| periodSeconds | int | 10 | How often (in seconds) to perform the probe. |
| successThreshold | int | 0 | Minimum consecutive successes for the probe to be considered successful after having failed. |
| failureThreshold | int | 3 |  Minimum consecutive failures for the probe to be considered failed after having succeeded. |
| exec | array | {} |  Exec specifies the action to take. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/#execaction-v1-core) |

> Note: To configure readiness probes, see {{% xref "/docs/one-eye/logging-operator/operation/readiness-probe.md" %}}.
