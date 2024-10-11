---
title: GELF
weight: 200
generated_file: true
---

# [GELF Output](https://github.com/hotschedules/fluent-plugin-gelf-hs)
## Overview
Fluentd output plugin for GELF. For details, see [https://github.com/bmichalkiewicz/fluent-plugin-gelf-best](https://github.com/bmichalkiewicz/fluent-plugin-gelf-best).

## Example
```yaml
spec:
  gelf:
    host: gelf-host
    port: 12201
```


## Configuration
## Output Config

### buffer (*Buffer, optional) {#output config-buffer}

Available since ghcr.io/kube-logging/fluentd:v1.16-full-build.139 [Buffer](../buffer/) 


### host (string, required) {#output config-host}

Destination host 


### max_bytes (int, optional) {#output config-max_bytes}

MaxBytes specifies the maximum size, in bytes, of each individual log message. For details, see [https://github.com/Graylog2/graylog2-server/issues/873](https://github.com/Graylog2/graylog2-server/issues/873) Available since ghcr.io/kube-logging/fluentd:v1.16-4.10-full

Default: 3200

### port (int, required) {#output-config-port}

Destination host port 


### protocol (string, optional) {#output-config-protocol}

Transport Protocol

Default: "udp"

### tls (*bool, optional) {#output-config-tls}

Enable TlS

Default: false

### tls_options (map[string]string, optional) {#output-config-tls_options}

TLS Options. For details, see [https://github.com/graylog-labs/gelf-rb/blob/72916932b789f7a6768c3cdd6ab69a3c942dbcef/lib/gelf/transport/tcp_tls.rb#L7-L12](https://github.com/graylog-labs/gelf-rb/blob/72916932b789f7a6768c3cdd6ab69a3c942dbcef/lib/gelf/transport/tcp_tls.rb#L7-L12).

Default: {}




## Example `GELF` output configurations

{{< highlight yaml >}}
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: gelf-output-sample
spec:
  gelf:
    host: gelf-host
    port: 12201
{{</ highlight >}}

Fluentd config result:

{{< highlight xml >}}
<match **>
	@type gelf
	@id test_gelf
	host gelf-host
	port 12201
</match>
{{</ highlight >}}


---
