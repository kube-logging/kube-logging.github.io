---
title: Azure Storage
weight: 200
generated_file: true
---

# Azure Storage output plugin for Fluentd
## Overview
Azure Storage output plugin buffers logs in local file and upload them to Azure Storage periodically.
More info at https://github.com/microsoft/fluent-plugin-azure-storage-append-blob

## Configuration
## Output Config

### path (string, optional) {#output config-path}

Path prefix of the files on Azure<br>

Default: -

### azure_storage_account (*secret.Secret, required) {#output config-azure_storage_account}

Your azure storage account<br>[Secret](../secret/)<br>

Default: -

### azure_storage_access_key (*secret.Secret, optional) {#output config-azure_storage_access_key}

Your azure storage access key<br>[Secret](../secret/)<br>

Default: -

### azure_storage_sas_token (*secret.Secret, optional) {#output config-azure_storage_sas_token}

Your azure storage sas token<br>[Secret](../secret/)<br>

Default: -

### azure_container (string, required) {#output config-azure_container}

Your azure storage container<br>

Default: -

### azure_imds_api_version (string, optional) {#output config-azure_imds_api_version}

Azure Instance Metadata Service API Version<br>

Default: -

### azure_object_key_format (string, optional) {#output config-azure_object_key_format}

Object key format <br>

Default:  %{path}%{time_slice}_%{index}.%{file_extension}

### auto_create_container (bool, optional) {#output config-auto_create_container}

Automatically create container if not exists<br>

Default:  true

### format (string, optional) {#output config-format}

Compat format type: out_file, json, ltsv (default: out_file)<br>

Default: json

### buffer (*Buffer, optional) {#output config-buffer}

[Buffer](../buffer/)<br>

Default: -


