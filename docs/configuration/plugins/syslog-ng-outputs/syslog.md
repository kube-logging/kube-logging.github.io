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

### host (string, required) {#syslogoutput-host}

Default: -

### port (int, optional) {#syslogoutput-port}

Default: -

### transport (string, optional) {#syslogoutput-transport}

Default: -

### close_on_input (*bool, optional) {#syslogoutput-close_on_input}

Default: -

### flags ([]string, optional) {#syslogoutput-flags}

Default: -

### flush_lines (int, optional) {#syslogoutput-flush_lines}

Default: -

### so_keepalive (*bool, optional) {#syslogoutput-so_keepalive}

Default: -

### suppress (int, optional) {#syslogoutput-suppress}

Default: -

### template (string, optional) {#syslogoutput-template}

Default: -

### template_escape (*bool, optional) {#syslogoutput-template_escape}

Default: -

### tls (*TLS, optional) {#syslogoutput-tls}

Default: -

### ts_format (string, optional) {#syslogoutput-ts_format}

Default: -

### disk_buffer (*DiskBuffer, optional) {#syslogoutput-disk_buffer}

Default: -


## TLS

### ca_dir (*secret.Secret, optional) {#tls-ca_dir}

Default: -

### ca_file (*secret.Secret, optional) {#tls-ca_file}

Default: -

### key_file (*secret.Secret, optional) {#tls-key_file}

Default: -

### cert_file (*secret.Secret, optional) {#tls-cert_file}

Default: -

### use-system-cert-store (*bool, optional) {#tls-use-system-cert-store}

Default: -


