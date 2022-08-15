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

The following example also configures disk-based buffering for the output (see the `disk_buffer` section):

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGOutput
metadata:
  name: test
  namespace: default
spec:
  syslog:
    host: 10.20.9.89
    port: 601
    disk_buffer:
      disk_buf_size: 512000000
      dir: /buffer
      reliable: true
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

For details on the available options of the output, see the [syslog-ng documentation](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/56#TOPIC-1829124).
