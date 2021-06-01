---
title: Http
weight: 200
generated_file: true
---

# Redis plugin for Fluentd
## Overview
 Sends logs to Redis endpoints.
 More info at https://github.com/fluent-plugins-nursery/fluent-plugin-redis

 #### Example output configurations
 ```
 spec:
   redis:
     host: redis-master.prod.svc.cluster.local
     buffer:
       tags: "[]"
       flush_interval: 10s
 ```

## Configuration
## Output Config

### host (string, optional) {#output config-host}

Host Redis endpoint <br>

Default:  localhost

### port (int, optional) {#output config-port}

Port of the Redis server <br>

Default:  6379

### db_number (int, optional) {#output config-db_number}

DbNumber database number is optional. <br>

Default:  0

### password (*secret.Secret, optional) {#output config-password}

Redis Server password<br>

Default: -

### insert_key_prefix (string, optional) {#output config-insert_key_prefix}

insert_key_prefix <br>

Default:  "${tag}"

### strftime_format (string, optional) {#output config-strftime_format}

strftime_format Users can set strftime format. <br>

Default:  "%s"

### allow_duplicate_key (bool, optional) {#output config-allow_duplicate_key}

allow_duplicate_key Allow insert key duplicate. It will work as update values. <br>

Default:  false

### ttl (int, optional) {#output config-ttl}

ttl If 0 or negative value is set, ttl is not set in each key.<br>

Default: -

### format (*Format, optional) {#output config-format}

[Format](../format/)<br>

Default: -

### buffer (*Buffer, optional) {#output config-buffer}

[Buffer](../buffer/)<br>

Default: -


