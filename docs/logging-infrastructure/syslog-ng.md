---
title: Configure syslog-ng
shorttitle: syslog-ng log forwarder
weight: 300
---

You can configure the deployment of the syslog-ng log forwarder via the **syslogNG** section of the {{% xref "/docs/logging-operator/logging-infrastructure/logging.md" %}}. For the detailed list of available parameters, see {{% xref "/docs/logging-operator/configuration/crds/v1beta1/syslogng_types.md" %}}.

The following example sets a volume mount that syslog-ng can use for buffering messages on the disk (if {{% xref "/docs/logging-operator/configuration/plugins/syslog-ng-outputs/disk_buffer.md" %}} is configured in the output).

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

{{< include-headless "cpu-memory-requirements.md" "logging-operator" >}}
