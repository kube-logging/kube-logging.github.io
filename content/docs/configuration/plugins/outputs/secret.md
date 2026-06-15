---
title: Secret definition
weight: 900
---

## Define secret value

Secrets can be used in logging-operator `Output` definitions.

> Secrets *MUST* be in the *SAME* namespace as the `Output` or `ClusterOutput` custom resource

**Example secret definition**
```yaml
aws_key_id:
  valueFrom:
    secretKeyRef:
      name: <kubernetes-secret-name>
      key: <kubernetes-secret-key>
```

For debug purposes you can define secret values directly. However this is *NOT* recommended in production.
```yaml
aws_key_id:
  value: "secretvalue"
```

Before rendering a secret value into the Fluentd configuration, the operator trims a single trailing newline or carriage return. As a result, a secret stored with a trailing newline (a common result of `echo "secret" | base64`) renders the same as one stored without it, so it won't break authentication for outputs. Newlines within the value are preserved, but the value is quoted and escaped so it can't inject additional Fluentd configuration directives.

## Define secret mount

There are cases when you can't inject secret into the configuration because the plugin need a file to read from. For this cases you can use `mountFrom`.

```yaml
tls_cert_path:
  mountFrom:
    secretKeyRef:
      name: <kubernetes-secret-name>
      key: <kubernetes-secret-key>
```

The operator will collect the secret and copy it to the `fluentd-output` secret. The fluentd configuration will contain the secret path.

**Example rendered configuration**
```
<match **>
    @type forward
    tls_cert_path /fluentd/etc/secret/default-fluentd-tls-tls.crt
    ...
</match>
```

### How it works?
Behind the scene the operator marks the secret with an annotation and watches it for changes as long as the annotation is present.

**Example annotated secret**
```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  annotations:
    logging.banzaicloud.io/default: watched
  name: fluentd-tls
  namespace: default
data:
  tls.crt: SGVsbG8gV29ybGQ=
```

> The annotation format is `logging.banzaicloud.io/<loggingRef>: watched`. Since the `name` part of the an annotation can't be empty the `default` applies to empty `loggingRef` value as well.

The mount path is generated from the secret information
```bash
/fluentd/etc/secret/$namespace-$secret_name-$secret_key
```
