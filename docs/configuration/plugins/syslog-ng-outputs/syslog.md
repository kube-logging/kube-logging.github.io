---
title: Syslog
weight: 200
---

The `syslog` output sends log records over a socket using the Syslog protocol (RFC 5424).

```yaml
  spec:
    syslog:
      host: 10.12.34.56
      transport: tls
      tls:
        ca_file:
          mountFrom:
            secretKeyRef:
              name: tls-secret
              key: ca.crt
        cert_file:
          mountFrom:
            secretKeyRef:
              name: tls-secret
              key: tls.crt
        key_file:
          mountFrom:
            secretKeyRef:
              name: tls-secret
              key: tls.key
```
