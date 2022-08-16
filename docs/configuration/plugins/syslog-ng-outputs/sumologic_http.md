---
title: sumologic-http
weight: 200
---

The `sumologic-http` output sends log records over HTTP to Sumo Logic.

## Prerequisites

You need a Sumo Logic account to use this output. For details, see the [syslog-ng documentation](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/55#TOPIC-1829118).

## Parameters

```yaml
  body: # Body content template to send
  deployment: # Deployment code for sumologic. More info: https://help.sumologic.com/APIs/General-API-Information/Sumo-Logic-Endpoints-by-Deployment-and-Firewall-Security
  collector: # Sumo Logic service token (secret)
  headers: # Extra headers for Sumologic like X-Sumo-Name
  tls: # Required TLS configuration for Sumologic. Minimal config is use-system-cert-store: true
  disk_buffer: # Disk buffer parameters
  batch-lines: # Collect messages into batches number of lines (recommended)
  batch-bytes: # Collect messages into batches size of batch 
  batch-timeout: # Time out for sending batch if no input available
```

## Example

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGOutput
metadata:
  name: test-sumo
  namespace: default
spec:
  sumologic-http:
    batch-lines: 1000
    disk_buffer:
      disk_buf_size: 512000000
      dir: /buffers
      reliable: true
    body: "$(format-json
                --subkeys json.
                --exclude json.kubernetes.annotations.*
                json.kubernetes.annotations=literal($(format-flat-json --subkeys json.kubernetes.annotations.))
                --exclude json.kubernetes.labels.*
                json.kubernetes.labels=literal($(format-flat-json --subkeys json.kubernetes.labels.)))"
    collector:
      valueFrom:
        secretKeyRef:
          key: token
          name: sumo-collector
    deployment: us2
    headers:
    - 'X-Sumo-Name: source-name'
    - 'X-Sumo-Category: source-category'
    tls:
      use-system-cert-store: true
```
