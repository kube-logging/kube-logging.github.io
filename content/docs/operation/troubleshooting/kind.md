---
title: Running on KinD
shorttitle: KinD
weight: 400
---

Persistent Volumes do not respect the `fsGroup` value on Kind so disable using a PVC for fluentd:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: example-on-kind
spec:
  fluentd:
    disablePvc: true
```

{{< include-headless "support-troubleshooting.md" "one-eye/logging-operator" >}}
