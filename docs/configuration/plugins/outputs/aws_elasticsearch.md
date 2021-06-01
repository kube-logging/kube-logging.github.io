---
title: Amazon Elasticsearch
weight: 200
generated_file: true
---

# Amazon Elasticsearch output plugin for Fluentd
## Overview
  More info at https://github.com/atomita/fluent-plugin-aws-elasticsearch-service

 #### Example output configurations
 ```
 spec:
   kinesisStream:
     stream_name: example-stream-name
     region: us-east-1
     format:
       type: json
 ```

## Configuration
## Amazon Elasticsearch

Send your logs to a Amazon Elasticsearch Service

### logstash_format (bool, optional) {#amazon elasticsearch-logstash_format}

logstash_format<br>

Default: -

### logstash_prefix (string, optional) {#amazon elasticsearch-logstash_prefix}

logstash_prefix<br>

Default: -

### include_tag_key (bool, optional) {#amazon elasticsearch-include_tag_key}

include_tag_key<br>

Default: -

### tag_key (string, optional) {#amazon elasticsearch-tag_key}

tag_key<br>

Default: -

### flush_interval (string, optional) {#amazon elasticsearch-flush_interval}

flush_interval<br>

Default: -

### endpoint (*EndpointCredentials, optional) {#amazon elasticsearch-endpoint}

AWS Endpoint Credentials<br>

Default: -

### format (*Format, optional) {#amazon elasticsearch-format}

[Format](../format/)<br>

Default: -

### buffer (*Buffer, optional) {#amazon elasticsearch-buffer}

[Buffer](../buffer/)<br>

Default: -


## Endpoint Credentials

endpoint

### region (string, optional) {#endpoint credentials-region}

AWS region. It should be in form like us-east-1, us-west-2. Default nil, which means try to find from environment variable AWS_REGION.<br>

Default: -

### url (string, optional) {#endpoint credentials-url}

AWS connection url.<br>

Default: -

### access_key_id (*secret.Secret, optional) {#endpoint credentials-access_key_id}

AWS access key id. This parameter is required when your agent is not running on EC2 instance with an IAM Role.<br>

Default: -

### secret_access_key (*secret.Secret, optional) {#endpoint credentials-secret_access_key}

AWS secret key. This parameter is required when your agent is not running on EC2 instance with an IAM Role.<br>

Default: -

### assume_role_arn (*secret.Secret, optional) {#endpoint credentials-assume_role_arn}

Typically, you can use AssumeRole for cross-account access or federation.<br>

Default: -

### ecs_container_credentials_relative_uri (*secret.Secret, optional) {#endpoint credentials-ecs_container_credentials_relative_uri}

Set with AWS_CONTAINER_CREDENTIALS_RELATIVE_URI environment variable value<br>

Default: -

### assume_role_session_name (*secret.Secret, optional) {#endpoint credentials-assume_role_session_name}

AssumeRoleWithWebIdentity https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html<br>

Default: -

### assume_role_web_identity_token_file (*secret.Secret, optional) {#endpoint credentials-assume_role_web_identity_token_file}

AssumeRoleWithWebIdentity https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html<br>

Default: -

### sts_credentials_region (*secret.Secret, optional) {#endpoint credentials-sts_credentials_region}

By default, the AWS Security Token Service (AWS STS) is available as a global service, and all AWS STS requests go to a single endpoint at https://sts.amazonaws.com. AWS recommends using Regional AWS STS endpoints instead of the global endpoint to reduce latency, build in redundancy, and increase session token validity. https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html<br>

Default: -


