---
title: TLS encryption
weight: 800
aliases:
    - /docs/one-eye/logging-operator/configuration/tls/
---

To use TLS encryption in your logging infrastructure, you have to configure encryption:

- for the log collection part of your logging pipeline (between Fluent Bit and Fluentd or Fluent bit and syslog-ng), and
- for the output plugin (between Fluentd or syslog-ng and the output backend).

For configuring the output, see the documentation of the output plugin you want to use at {{% xref "/docs/configuration/plugins/outputs/_index.md" %}}.

For Fluentd and Fluent Bit, you can configure encryption in the `logging` resource using the following parameters:

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

## Customize the Fluentd forward input TLS settings

When `fluentd.tls.enabled` is `true`, the Logging operator configures TLS on the forward input that receives logs from Fluent Bit. To meet a TLS policy, such as a restricted cipher list, set the other TLS options of that listener in the `fluentd.forwardInputConfig.transport` section. For the available options, see {{% xref "/docs/configuration/plugins/common/transport.md" %}}.

The operator always sets the following fields and overrides any value you set for them:

- `ca_path`, `cert_path`, and `private_key_path`, which point to the files of the secret set in `secretName`
- `client_cert_auth`, which is always `true`
- the `security` section, which uses the value of `sharedKey`

The operator keeps the other fields of the `transport` section, such as `ciphers`, `version`, and `protocol`. If you don't set `version`, it defaults to `TLSv1_2`.

For example, to restrict the ciphers that the Fluentd forward input accepts:

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
    forwardInputConfig:
      transport:
        ciphers: "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384"
  controlNamespace: logging
```

For other parameters of the logging resource, see {{% xref "/docs/configuration/crds/v1beta1/logging_types.md" %}}.
