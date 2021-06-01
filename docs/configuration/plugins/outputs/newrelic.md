---
title: NewRelic
weight: 200
generated_file: true
---

# New Relic Logs plugin for Fluentd
## Overview
**newrelic** output plugin send log data to New Relic Logs

 #### Example output configurations
 ```
 spec:
   newrelic:
     license_key:
       valueFrom:
         secretKeyRef:
           name: logging-newrelic
           key: licenseKey
 ```

## Configuration
## Output Config

### api_key (*secret.Secret, optional) {#output config-api_key}

New Relic API Insert key<br>[Secret](../secret/)<br>

Default: -

### license_key (*secret.Secret, optional) {#output config-license_key}

New Relic License Key (recommended)<br>[Secret](../secret/"<br>LicenseKey *secret.Secret `json:"license_key)`<br>

Default: -

### base_uri (string, optional) {#output config-base_uri}

New Relic ingestion endpoint<br>[Secret](../secret/)<br>

Default: https://log-api.newrelic.com/log/v1


