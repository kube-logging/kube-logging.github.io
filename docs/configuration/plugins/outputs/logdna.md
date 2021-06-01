---
title: LogDNA
weight: 200
generated_file: true
---

# [LogDNA Output](https://github.com/logdna/fluent-plugin-logdna)
## Overview
 This plugin has been designed to output logs to LogDNA. Example Deployment: [Transport Nginx Access Logs into LogDNA with Logging Operator](https://raw.githubusercontent.com/banzaicloud/logging-operator/master/docs/examples/logging_output_logdna.yaml)

## Configuration
## LogDNA

Send your logs to LogDNA

### api_key (string, required) {#logdna-api_key}

LogDNA Api key<br>

Default: -

### hostname (string, required) {#logdna-hostname}

Hostname<br>

Default: -

### app (string, optional) {#logdna-app}

Application name<br>

Default: -

### tags (string, optional) {#logdna-tags}

Comma-Separated List of Tags, Optional<br>

Default: -

### request_timeout (string, optional) {#logdna-request_timeout}

HTTPS POST Request Timeout, Optional. Supports s and ms Suffices <br>

Default:  30 s

### ingester_domain (string, optional) {#logdna-ingester_domain}

Custom Ingester URL, Optional <br>

Default:  https://logs.logdna.com

### ingester_endpoint (string, optional) {#logdna-ingester_endpoint}

Custom Ingester Endpoint, Optional <br>

Default:  /logs/ingest

### buffer (*Buffer, optional) {#logdna-buffer}

[Buffer](../buffer/)<br>

Default: -


 #### Example `LogDNA` filter configurations
 ```
 apiVersion: logging.banzaicloud.io/v1beta1
 kind: Output
 metadata:
   name: logdna-output-sample
 spec:
   logdna:
     api_key: xxxxxxxxxxxxxxxxxxxxxxxxxxx
     hostname: logging-operator
     app: my-app
     tags: web,dev
     ingester_domain https://logs.logdna.com
     ingester_endpoint /logs/ingest
 ```

 #### Fluentd Config Result
 ```
<match **>
	@type logdna
	@id test_logdna
	api_key xxxxxxxxxxxxxxxxxxxxxxxxxxy
	app my-app
	hostname logging-operator
</match>
 ```

---
