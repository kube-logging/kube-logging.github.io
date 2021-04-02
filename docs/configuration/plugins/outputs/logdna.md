---
title: LogDNA
weight: 200
generated_file: true
---

# [LogDNA Output](https://github.com/logdna/fluent-plugin-logdna)
## Overview
 This plugin has been designed to output logs to LogDNA. Example Deployment: [Transport Nginx Access Logs into LogDNA with Logging Operator](https://raw.githubusercontent.com/banzaicloud/logging-operator/master/docs/examples/logging_output_logdna.yaml)

## Configuration
### LogDNA
#### Send your logs to LogDNA

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| api_key | string | Yes | - | LogDNA Api key<br> |
| hostname | string | Yes | - | Hostname<br> |
| app | string | No | - | Application name<br> |
| tags | string | No | - | Comma-Separated List of Tags, Optional<br> |
| request_timeout | string | No |  30 s | HTTPS POST Request Timeout, Optional. Supports s and ms Suffices <br> |
| ingester_domain | string | No |  https://logs.logdna.com | Custom Ingester URL, Optional <br> |
| ingester_endpoint | string | No |  /logs/ingest | Custom Ingester Endpoint, Optional <br> |
| buffer | *Buffer | No | - | [Buffer](../buffer/)<br> |
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
