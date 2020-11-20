---
title: Format rfc5424
weight: 200
generated_file: true
---

### FormatRfc5424
| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| type | string | No |  json | Output line formatting: out_file,json,ltsv,csv,msgpack,hash,single_value <br> |
| rfc6587_message_size | *bool | No |  true | Prepends message length for syslog transmission <br> |
| hostname_field | string | No |  hostname | Sets host name in syslog from field in fluentd, delimited by '.' <br> |
| app_name_field | string | No |  app_name | Sets app name in syslog from field in fluentd, delimited by '.' <br> |
| proc_id_field | string | No |  proc_id | Sets proc id in syslog from field in fluentd, delimited by '.'  <br> |
| message_id_field | string | No |  message_id | Sets msg id in syslog from field in fluentd, delimited by '.' <br> |
| structured_data_field | string | No | - | Sets structured data in syslog from field in fluentd, delimited by '.' (default structured_data)<br> |
| log_field | string | No |  log | Sets log in syslog from field in fluentd, delimited by '.' <br> |
