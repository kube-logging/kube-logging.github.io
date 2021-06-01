---
title: SumoLogic
weight: 200
generated_file: true
---

# SumoLogic output plugin for Fluentd
## Overview
This plugin has been designed to output logs or metrics to SumoLogic via a HTTP collector endpoint
More info at https://github.com/SumoLogic/fluentd-output-sumologic

 Example secret for HTTP input URL
 ```
export URL='https://endpoint1.collection.eu.sumologic.com/receiver/v1/http/.......'
kubectl create secret generic sumo-output --from-literal "endpoint=$URL"
```

 Example ClusterOutput

```
apiVersion: logging.banzaicloud.io/v1beta1
kind: ClusterOutput
metadata:
  name: sumo-output
spec:
  sumologic:
    buffer:
      flush_interval: 10s
      flush_mode: interval
    compress: true
    endpoint:
      valueFrom:
        secretKeyRef:
          key: endpoint
          name: sumo-output
    source_name: test1
```

## Configuration
## Output Config

### data_type (string, optional) {#output config-data_type}

The type of data that will be sent to Sumo Logic, either logs or metrics <br>

Default:  logs

### endpoint (*secret.Secret, required) {#output config-endpoint}

SumoLogic HTTP Collector URL<br>

Default: -

### verify_ssl (bool, optional) {#output config-verify_ssl}

Verify ssl certificate. <br>

Default:  true

### metric_data_format (string, optional) {#output config-metric_data_format}

The format of metrics you will be sending, either graphite or carbon2 or prometheus <br>

Default:  graphite

### log_format (string, optional) {#output config-log_format}

Format to post logs into Sumo. <br>

Default:  json

### log_key (string, optional) {#output config-log_key}

Used to specify the key when merging json or sending logs in text format <br>

Default:  message

### source_category (string, optional) {#output config-source_category}

Set _sourceCategory metadata field within SumoLogic <br>

Default:  nil

### source_name (string, required) {#output config-source_name}

Set _sourceName metadata field within SumoLogic - overrides source_name_key (default is nil)<br>

Default: -

### source_name_key (string, optional) {#output config-source_name_key}

Set as source::path_key's value so that the source_name can be extracted from Fluentd's buffer <br>

Default:  source_name

### source_host (string, optional) {#output config-source_host}

Set _sourceHost metadata field within SumoLogic <br>

Default:  nil

### open_timeout (int, optional) {#output config-open_timeout}

Set timeout seconds to wait until connection is opened. <br>

Default:  60

### add_timestamp (bool, optional) {#output config-add_timestamp}

Add timestamp (or timestamp_key) field to logs before sending to sumologic <br>

Default:  true

### timestamp_key (string, optional) {#output config-timestamp_key}

Field name when add_timestamp is on <br>

Default:  timestamp

### proxy_uri (string, optional) {#output config-proxy_uri}

Add the uri of the proxy environment if present.<br>

Default: -

### disable_cookies (bool, optional) {#output config-disable_cookies}

Option to disable cookies on the HTTP Client. <br>

Default:  false

### delimiter (string, optional) {#output config-delimiter}

Delimiter <br>

Default:  .

### custom_fields ([]string, optional) {#output config-custom_fields}

Comma-separated key=value list of fields to apply to every log. [more information](https://help.sumologic.com/Manage/Fields#http-source-fields)<br>

Default: -

### sumo_client (string, optional) {#output config-sumo_client}

Name of sumo client which is send as X-Sumo-Client header <br>

Default:  fluentd-output

### compress (*bool, optional) {#output config-compress}

Compress payload <br>

Default:  false

### compress_encoding (string, optional) {#output config-compress_encoding}

Encoding method of compression (either gzip or deflate) <br>

Default:  gzip

### custom_dimensions (string, optional) {#output config-custom_dimensions}

Dimensions string (eg "cluster=payment, service=credit_card") which is going to be added to every metric record.<br>

Default: -

### buffer (*Buffer, optional) {#output config-buffer}

[Buffer](../buffer/)<br>

Default: -


