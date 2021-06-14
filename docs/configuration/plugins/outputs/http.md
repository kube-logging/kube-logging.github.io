---
title: Http
weight: 200
generated_file: true
---

# Http plugin for Fluentd
## Overview
 Sends logs to HTTP/HTTPS endpoints.
 More info at https://docs.fluentd.org/output/http.

 #### Example output configurations
 ```
 spec:
   http:
     endpoint: http://logserver.com:9000/api
     buffer:
       tags: "[]"
       flush_interval: 10s
 ```

## Configuration
## Output Config

### endpoint (string, required) {#output config-endpoint}

Endpoint for HTTP request.<br>

Default: -

### http_method (string, optional) {#output config-http_method}

Method for HTTP request. [post, put] <br>

Default:  post

### proxy (string, optional) {#output config-proxy}

Proxy for HTTP request.<br>

Default: -

### content_type (string, optional) {#output config-content_type}

Content-Profile for HTTP request.<br>

Default: -

### json_array (bool, optional) {#output config-json_array}

Using array format of JSON. This parameter is used and valid only for json format. When json_array as true, Content-Profile should be application/json and be able to use JSON data for the HTTP request body.  <br>

Default:  false

### format (*Format, optional) {#output config-format}

[Format](../format/)<br>

Default: -

### headers (map[string]string, optional) {#output config-headers}

Additional headers for HTTP request.<br>

Default: -

### open_timeout (int, optional) {#output config-open_timeout}

Connection open timeout in seconds.<br>

Default: -

### read_timeout (int, optional) {#output config-read_timeout}

Read timeout in seconds.<br>

Default: -

### ssl_timeout (int, optional) {#output config-ssl_timeout}

TLS timeout in seconds.<br>

Default: -

### tls_version (string, optional) {#output config-tls_version}

The default version of TLS transport. [TLSv1_1, TLSv1_2] <br>

Default:  TLSv1_2

### tls_ciphers (string, optional) {#output config-tls_ciphers}

The cipher configuration of TLS transport. <br>

Default:  ALL:!aNULL:!eNULL:!SSLv2

### tls_ca_cert_path (*secret.Secret, optional) {#output config-tls_ca_cert_path}

The CA certificate path for TLS.<br>

Default: -

### tls_client_cert_path (*secret.Secret, optional) {#output config-tls_client_cert_path}

The client certificate path for TLS.<br>

Default: -

### tls_private_key_path (*secret.Secret, optional) {#output config-tls_private_key_path}

The client private key path for TLS.<br>

Default: -

### tls_private_key_passphrase (*secret.Secret, optional) {#output config-tls_private_key_passphrase}

The client private key passphrase for TLS.<br>

Default: -

### tls_verify_mode (string, optional) {#output config-tls_verify_mode}

The verify mode of TLS. [peer, none] <br>

Default:  peer

### error_response_as_unrecoverable (*bool, optional) {#output config-error_response_as_unrecoverable}

Raise UnrecoverableError when the response code is non success, 1xx/3xx/4xx/5xx. If false, the plugin logs error message instead of raising UnrecoverableError. <br>

Default:  true

### retryable_response_codes ([]int, optional) {#output config-retryable_response_codes}

List of retryable response codes. If the response code is included in this list, the plugin retries the buffer flush. <br>

Default:  [503]

### auth (*HTTPAuth, optional) {#output config-auth}

[HTTP auth](#http-auth-config)<br>

Default: -

### buffer (*Buffer, optional) {#output config-buffer}

[Buffer](../buffer/)<br>

Default: -


## HTTP auth config

http_auth

### username (*secret.Secret, required) {#http auth config-username}

Username for basic authentication.<br>[Secret](../secret/)<br>

Default: -

### password (*secret.Secret, required) {#http auth config-password}

Password for basic authentication.<br>[Secret](../secret/)<br>

Default: -


