---
title: Syslog
weight: 200
generated_file: true
---

# [Syslog Output](https://github.com/cloudfoundry/fluent-plugin-syslog_rfc5424)
## Overview
 Fluentd output plugin for remote syslog with RFC5424 headers logs.

## Configuration
## SyslogOutputConfig

### host (string, required) {#syslogoutputconfig-host}

Destination host address<br>

Default: -

### port (int, optional) {#syslogoutputconfig-port}

Destination host port <br>

Default:  "514"

### transport (string, optional) {#syslogoutputconfig-transport}

Transport Protocol <br>

Default:  "tls"

### insecure (*bool, optional) {#syslogoutputconfig-insecure}

skip ssl validation <br>

Default:  false

### trusted_ca_path (*secret.Secret, optional) {#syslogoutputconfig-trusted_ca_path}

file path to ca to trust<br>

Default: -

### format (*FormatRfc5424, optional) {#syslogoutputconfig-format}

[Format](../format/)<br>

Default: -

### buffer (*Buffer, optional) {#syslogoutputconfig-buffer}

[Buffer](../buffer/)<br>

Default: -


 #### Example `File` output configurations
 ```
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: demo-output
spec:
  syslog:
    host: SYSLOG-HOST
    port: 123
    format:
      app_name_field: example.custom_field_1
      proc_id_field: example.custom_field_2
    buffer:
      timekey: 1m
      timekey_wait: 10s
      timekey_use_utc: true
 ```

 #### Fluentd Config Result
 ```
  <match **>
	@type syslog_rfc5424
	@id test_syslog
	host SYSLOG-HOST
	port 123
  <format>
    @type syslog_rfc5424
    app_name_field example.custom_field_1
    proc_id_field example.custom_field_2
  </format>
	<buffer tag,time>
	  @type file
	  path /buffers/test_file.*.buffer
	  retry_forever true
	  timekey 1m
	  timekey_use_utc true
	  timekey_wait 30s
	</buffer>
  </match>
 ```

---
