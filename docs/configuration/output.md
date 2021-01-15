---
title: Output and ClusterOutput
weight: 50
---

The `Output` resource defines an output where your Flows can send the log messages. The output is a `namespaced` resource which means only a `Flow` within the *same* namespace can access it. You can use `secrets` in these definitions, but they must also be in the same namespace.
Outputs are the final stage for a `logging flow`. You can define multiple `outputs` and attach them to multiple `flows`.

`ClusterOutput` defines an Output **without** namespace restrictions. It is only evaluated in the `controlNamespace` by default unless `allowClusterResourcesFromAllNamespaces` is set to true.

> Note: `Flow` can be connected to `Output` and `ClusterOutput`, but `ClusterFlow` can be attached only to `ClusterOutput`.

- For the details of the supported output plugins, see {{% xref "/docs/one-eye/logging-operator/configuration/plugins/outputs/_index.md" %}}.
- For the details of `Output` custom resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/output_types.md" %}}.
- For the details of `ClusterOutput` custom resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/clusteroutput_types.md" %}}.

## S3 output example

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
          namespace: default
    aws_sec_key:
      valueFrom:
        secretKeyRef:
          name: s3-secret
          key: awsSecretAccessKey
          namespace: default
    s3_bucket: example-logging-bucket
    s3_region: eu-west-1
    path: logs/${tag}/%Y/%m/%d/
    buffer:
      timekey: 1m
      timekey_wait: 10s
      timekey_use_utc: true
```
