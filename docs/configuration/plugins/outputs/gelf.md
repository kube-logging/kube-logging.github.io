---
title: GELF
weight: 200
generated_file: true
---

# [GELF Output](https://github.com/hotschedules/fluent-plugin-gelf-hs)
## Overview
 Fluentd output plugin for GELF.

## Configuration
### Output Config
| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| host | string | Yes | - | Destination host<br> |
| port | int | Yes | - | Destination host port<br> |
| protocol | string | No |  "udp" | Transport Protocol <br> |
| tls | *bool | No |  false | Enable TlS <br> |
| tls_options | map[string]string | No |  {} | TLS Options  - for options see https://github.com/graylog-labs/gelf-rb/blob/72916932b789f7a6768c3cdd6ab69a3c942dbcef/lib/gelf/transport/tcp_tls.rb#L7-L12<br> |
| buffer | *Buffer | No | - | [Buffer](../buffer/)<br> |

 #### Example `GELF` output configurations
 ```
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: gelf-output-sample
spec:
  gelf:
    host: gelf-host
    port: 12201
    buffer:
      flush_thread_count: 8
      flush_interval: 5s
      chunk_limit_size: 8M
      queue_limit_length: 512
      retry_max_interval: 30
      retry_forever: true
 ```

 #### Fluentd Config Result
 ```
  <match **>
	@type gelf
	@id test_gelf
	host gelf-host
	port 12201
	<buffer tag,time>
	  @type file
	  path /buffers/test_file.*.buffer
    flush_thread_count 8
    flush_interval 5s
    chunk_limit_size 8M
    queue_limit_length 512
    retry_max_interval 30
    retry_forever true
	</buffer>
  </match>
 ```

---
