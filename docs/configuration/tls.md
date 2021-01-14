---
title: TLS encryption
weight: 200
---

To use TLS encryption in your logging infrastructure, you have to configure encryption:

- for the log collection part of your logging pipeline (between FLuent-bit and Fluentd), and
- for the output plugin (between Fluentd and the output backend).

For configuring the output, see the documentation of the output plugin you want to use at {{% xref "/docs/one-eye/logging-operator/configuration/plugins/outputs/_index.md" %}}.

For Fluentd and Fluent-bit, you can configure encryption in the `logging` resource using the following parameters:

| Name                    | Type           | Default | Description |
|-------------------------|----------------|---------|-------------|
| enabled | bool | "Yes" | Enable TLS encryption |
| secretName | string | "" | Kubernetes secret that contains: **tls.crt, tls.key, ca.crt** |
| sharedKey | string | "" | Shared secret for fluentd authentication |

For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-tls
spec:
  fluentd:
    tls:
      enabled: true
      secretName: fluentd-tls
      sharedKey: example-secret
  fluentbit:
    tls:
      enabled: true
      secretName: fluentbit-tls
      sharedKey: example-secret
  controlNamespace: logging
```

For other parameters of the logging resource, see {{% xref "/docs/one-eye/logging-operator/configuration/crds/v1beta1/logging_types.md" %}}.
