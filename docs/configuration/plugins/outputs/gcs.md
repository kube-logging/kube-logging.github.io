---
title: Google Cloud Storage
weight: 200
generated_file: true
---

## GCSOutput

### project (string, required) {#gcsoutput-project}

Project identifier for GCS<br>

Default: -

### keyfile (string, optional) {#gcsoutput-keyfile}

Path of GCS service account credentials JSON file<br>

Default: -

### credentials_json (*secret.Secret, optional) {#gcsoutput-credentials_json}

GCS service account credentials in JSON format<br>[Secret](../secret/)<br>

Default: -

### client_retries (int, optional) {#gcsoutput-client_retries}

Number of times to retry requests on server error<br>

Default: -

### client_timeout (int, optional) {#gcsoutput-client_timeout}

Default timeout to use in requests<br>

Default: -

### bucket (string, required) {#gcsoutput-bucket}

Name of a GCS bucket<br>

Default: -

### object_key_format (string, optional) {#gcsoutput-object_key_format}

Format of GCS object keys <br>

Default:  %{path}%{time_slice}_%{index}.%{file_extension}

### path (string, optional) {#gcsoutput-path}

Path prefix of the files on GCS<br>

Default: -

### store_as (string, optional) {#gcsoutput-store_as}

Archive format on GCS: gzip json text <br>

Default:  gzip

### transcoding (bool, optional) {#gcsoutput-transcoding}

Enable the decompressive form of transcoding<br>

Default: -

### auto_create_bucket (bool, optional) {#gcsoutput-auto_create_bucket}

Create GCS bucket if it does not exists <br>

Default:  true

### hex_random_length (int, optional) {#gcsoutput-hex_random_length}

Max length of `%{hex_random}` placeholder(4-16) <br>

Default:  4

### overwrite (bool, optional) {#gcsoutput-overwrite}

Overwrite already existing path <br>

Default:  false

### acl (string, optional) {#gcsoutput-acl}

Permission for the object in GCS: auth_read owner_full owner_read private project_private public_read<br>

Default: -

### storage_class (string, optional) {#gcsoutput-storage_class}

Storage class of the file: dra nearline coldline multi_regional regional standard<br>

Default: -

### encryption_key (string, optional) {#gcsoutput-encryption_key}

Customer-supplied, AES-256 encryption key<br>

Default: -

### object_metadata ([]ObjectMetadata, optional) {#gcsoutput-object_metadata}

User provided web-safe keys and arbitrary string values that will returned with requests for the file as "x-goog-meta-" response headers.<br>[Object Metadata](#objectmetadata)<br>

Default: -

### format (*Format, optional) {#gcsoutput-format}

[Format](../format/)<br>

Default: -

### buffer (*Buffer, optional) {#gcsoutput-buffer}

[Buffer](../buffer/)<br>

Default: -


## ObjectMetadata

### key (string, required) {#objectmetadata-key}

Key<br>

Default: -

### value (string, required) {#objectmetadata-value}

Value<br>

Default: -


