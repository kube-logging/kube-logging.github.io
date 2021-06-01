---
title: Datadog
weight: 200
generated_file: true
---

# Datadog output plugin for Fluentd
## Overview
It mainly contains a proper JSON formatter and a socket handler that streams logs directly to Datadog - so no need to use a log shipper if you don't wan't to.
More info at https://github.com/DataDog/fluent-plugin-datadog

## Configuration
## Output Config

### api_key (*secret.Secret, required) {#output config-api_key}

This parameter is required in order to authenticate your fluent agent. <br>+docLink:"Secret,../secret/"<br>

Default:  nil

### use_json (bool, optional) {#output config-use_json}

Event format, if true, the event is sent in json format. Othwerwise, in plain text.  <br>

Default:  true

### include_tag_key (bool, optional) {#output config-include_tag_key}

Automatically include the Fluentd tag in the record.  <br>

Default:  false

### tag_key (string, optional) {#output config-tag_key}

Where to store the Fluentd tag. <br>

Default:  "tag"

### timestamp_key (string, optional) {#output config-timestamp_key}

Name of the attribute which will contain timestamp of the log event. If nil, timestamp attribute is not added. <br>

Default:  "@timestamp"

### use_ssl (bool, optional) {#output config-use_ssl}

If true, the agent initializes a secure connection to Datadog. In clear TCP otherwise.  <br>

Default:  true

### no_ssl_validation (bool, optional) {#output config-no_ssl_validation}

Disable SSL validation (useful for proxy forwarding)  <br>

Default:  false

### ssl_port (string, optional) {#output config-ssl_port}

Port used to send logs over a SSL encrypted connection to Datadog. If use_http is disabled, use 10516 for the US region and 443 for the EU region. <br>

Default:  "443"

### max_retries (string, optional) {#output config-max_retries}

The number of retries before the output plugin stops. Set to -1 for unlimited retries <br>

Default:  "-1"

### max_backoff (string, optional) {#output config-max_backoff}

The maximum time waited between each retry in seconds <br>

Default:  "30"

### use_http (bool, optional) {#output config-use_http}

Enable HTTP forwarding. If you disable it, make sure to change the port to 10514 or ssl_port to 10516  <br>

Default:  true

### use_compression (bool, optional) {#output config-use_compression}

Enable log compression for HTTP  <br>

Default:  true

### compression_level (string, optional) {#output config-compression_level}

Set the log compression level for HTTP (1 to 9, 9 being the best ratio) <br>

Default:  "6"

### dd_source (string, optional) {#output config-dd_source}

This tells Datadog what integration it is <br>

Default:  nil

### dd_sourcecategory (string, optional) {#output config-dd_sourcecategory}

Multiple value attribute. Can be used to refine the source attribute <br>

Default:  nil

### dd_tags (string, optional) {#output config-dd_tags}

Custom tags with the following format "key1:value1, key2:value2" <br>

Default:  nil

### dd_hostname (string, optional) {#output config-dd_hostname}

Used by Datadog to identify the host submitting the logs. <br>

Default:  "hostname -f"

### service (string, optional) {#output config-service}

Used by Datadog to correlate between logs, traces and metrics. <br>

Default:  nil

### port (string, optional) {#output config-port}

Proxy port when logs are not directly forwarded to Datadog and ssl is not used <br>

Default:  "80"

### host (string, optional) {#output config-host}

Proxy endpoint when logs are not directly forwarded to Datadog	 <br>

Default:  "http-intake.logs.datadoghq.com"

### buffer (*Buffer, optional) {#output config-buffer}

[Buffer](../buffer/)<br>

Default: -


