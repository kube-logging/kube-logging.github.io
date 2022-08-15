---
title: SyslogNGOutput and SyslogNGClusterOutput
weight: 150
---

The `SyslogNGOutput` resource defines an output where your SyslogNGFlows can send the log messages. The output is a `namespaced` resource which means only a `SyslogNGFlow` within the *same* namespace can access it. You can use `secrets` in these definitions, but they must also be in the same namespace.
Outputs are the final stage for a `logging flow`. You can define multiple `SyslogNGoutputs` and attach them to multiple `SyslogNGFlows`.

`SyslogNGClusterOutput` defines a SyslogNGOutput **without** namespace restrictions. It is only evaluated in the `controlNamespace` by default unless `allowClusterResourcesFromAllNamespaces` is set to true.

> Note: `SyslogNGFlow` can be connected to `SyslogNGOutput` and `SyslogNGClusterOutput`, but `SyslogNGClusterFlow` can be attached only to `SyslogNGClusterOutput`.

The following example defines a simple `SyslogNGOutput` resource that sends the logs to the specified syslog server using the RFC5424 Syslog protocol in a TLS-encrypted connection.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGOutput
metadata:
  name: syslog-output
  namespace: default
spec:
  syslog:
    host: 10.20.9.89
    port: 601
    template: "$(format-json
                --subkeys json.
                --exclude json.kubernetes.labels.*
                json.kubernetes.labels=literal($(format-flat-json --subkeys json.kubernetes.labels.)))\n"
    tls:
      ca_file:
        mountFrom:
          secretKeyRef:
            key: ca.crt
            name: syslog-tls-cert
      cert_file:
        mountFrom:
          secretKeyRef:
            key: tls.crt
            name: syslog-tls-cert
      key_file:
        mountFrom:
          secretKeyRef:
            key: tls.key
            name: syslog-tls-cert
    transport: tls
```

- For the details of the supported output plugins, see {{% xref "/docs/logging-operator/configuration/plugins/syslog-ng-outputs/_index.md" %}}.
- For the details of `SyslogNGOutput` custom resource, see {{% xref "/docs/logging-operator/configuration/crds/v1beta1/syslogng_output_types.md" %}}.
- For the details of `SyslogNGClusterOutput` custom resource, see {{% xref "/docs/logging-operator/configuration/crds/v1beta1/syslogng_clusteroutput_types.md" %}}.
