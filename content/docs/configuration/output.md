---
title: Output and ClusterOutput
weight: 50
---

Outputs are the destinations where your log forwarder sends the log messages, for example, to a file. Depending on which log forwarder you use, you have to configure different custom resources.

## Fluentd outputs

- The `Output` resource defines an output where your **Fluentd** Flows can send the log messages. The output is a `namespaced` resource which means only a `Flow` within the *same* namespace can access it. You can use `secrets` in these definitions, but they must also be in the same namespace.
Outputs are the final stage for a `logging flow`. You can define multiple `outputs` and attach them to multiple `flows`.
- `ClusterOutput` defines an Output **without** namespace restrictions. It is only evaluated in the `controlNamespace` by default unless `allowClusterResourcesFromAllNamespaces` is set to true.

> Note: `Flow` can be connected to `Output` and `ClusterOutput`, but `ClusterFlow` can be attached only to `ClusterOutput`.

- For the details of the supported output plugins, see {{% xref "/docs/configuration/plugins/outputs/_index.md" %}}.
- For the details of `Output` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/output_types.md" %}}.
- For the details of `ClusterOutput` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/clusteroutput_types.md" %}}.

### Fluentd S3 output example

The following snippet defines an Amazon S3 bucket as an output.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: s3-output-sample
spec:
  s3:
    aws_key_id:
      valueFrom:
        secretKeyRef:
          name: s3-secret
          key: awsAccessKeyId
    aws_sec_key:
      valueFrom:
        secretKeyRef:
          name: s3-secret
          key: awsSecretAccessKey
    s3_bucket: example-logging-bucket
    s3_region: eu-west-1
    path: logs/${tag}/%Y/%m/%d/
    buffer:
      timekey: 1m
      timekey_wait: 10s
      timekey_use_utc: true
```

## syslog-ng outputs {#syslogngoutput}

- The `SyslogNGOutput` resource defines an output for **syslog-ng** where your SyslogNGFlows can send the log messages. The output is a `namespaced` resource which means only a `SyslogNGFlow` within the *same* namespace can access it. You can use `secrets` in these definitions, but they must also be in the same namespace.
Outputs are the final stage for a `logging flow`. You can define multiple `SyslogNGOutputs` and attach them to multiple `SyslogNGFlows`.
- `SyslogNGClusterOutput` defines a SyslogNGOutput **without** namespace restrictions. It is only evaluated in the `controlNamespace` by default unless `allowClusterResourcesFromAllNamespaces` is set to true.

> Note: `SyslogNGFlow` can be connected to `SyslogNGOutput` and `SyslogNGClusterOutput`, but a `SyslogNGClusterFlow` can only be attached to a `SyslogNGClusterOutput`.

### RFC5424 syslog-ng output example

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

- For the details of the supported output plugins, see {{% xref "/docs/configuration/plugins/syslog-ng-outputs/_index.md" %}}.
- For the details of `SyslogNGOutput` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/syslogng_output_types.md" %}}.
- For the details of `SyslogNGClusterOutput` custom resource, see {{% xref "/docs/configuration/crds/v1beta1/syslogng_clusteroutput_types.md" %}}.

## protected flag cluster outputs

Since versions:

- 4.7 for Fluentd
- 5.0 for Syslog-ng

You can set the `protected` flag on a `ClusterOutput` and `SyslogNGClusterOutput`. This prevents namespaced `Flows` and `SyslogNGFlows` from sending logs to these outputs and only allows `ClusterFlows` and `SyslogNGClusterFlows` to reference it.

By default, `ClusterOutputs` can be referenced by any `Flow`. Setting the `protected` flag restricts this access to `ClusterFlows` only.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: ClusterOutput
metadata:
  name: protected-cluster-output
  namespace: infra
spec:
  protected: true
  s3:
    aws_key_id:
      valueFrom:
        secretKeyRef:
          name: s3-secret
          key: awsAccessKeyId
    aws_sec_key:
      valueFrom:
        secretKeyRef:
          name: s3-secret
          key: awsSecretAccessKey
    s3_bucket: example-logging-bucket
    s3_region: eu-west-1
    path: logs/${tag}/%Y/%m/%d/
    buffer:
      timekey: 1m
      timekey_wait: 10s
      timekey_use_utc: true
```
