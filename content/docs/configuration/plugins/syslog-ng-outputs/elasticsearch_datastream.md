---
title: Elasticsearch datastream
weight: 200
generated_file: true
---

## Overview

Based on the [ElasticSearch datastream destination of AxoSyslog](https://axoflow.com/docs/axosyslog-core/chapter-destinations/configuring-destinations-elasticsearch-datastream/).

Available in Logging operator version 4.9 and later.

## Example

{{< highlight yaml >}}
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGOutput
metadata:
  name: elasticsearch-datastream
spec:
  elasticsearch-datastream:
    url: "https://elastic-endpoint:9200/my-data-stream/_bulk"
    user: "username"
    password:
      valueFrom:
        secretKeyRef:
          name: elastic
          key: password
{{</ highlight >}}


## Configuration
## ElasticsearchDatastreamOutput

###  (HTTPOutput, required) {#elasticsearchdatastreamoutput-}


### disk_buffer (*DiskBuffer, optional) {#elasticsearchdatastreamoutput-disk_buffer}

This option enables putting outgoing messages into the disk buffer of the destination to avoid message loss in case of a system failure on the destination side. For details, see the [Syslog-ng DiskBuffer options](../disk_buffer/).

Default: false

### record (string, optional) {#elasticsearchdatastreamoutput-record}

Arguments to the `$format-json()` template function. Default: `"--scope rfc5424 --exclude DATE --key ISODATE @timestamp=${ISODATE}"` 



