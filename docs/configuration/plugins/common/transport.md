---
title: Transport
weight: 200
generated_file: true
---

## Transport

### protocol (string, optional) {#transport-protocol}

Protocol Default: :tcp<br>

Default: -

### version (string, optional) {#transport-version}

Version Default: 'TLSv1_2'<br>

Default: -

### ciphers (string, optional) {#transport-ciphers}

Ciphers Default: "ALL:!aNULL:!eNULL:!SSLv2"<br>

Default: -

### insecure (bool, optional) {#transport-insecure}

Use secure connection when use tls) Default: false<br>

Default: -

### ca_path (string, optional) {#transport-ca_path}

Specify path to CA certificate file<br>

Default: -

### cert_path (string, optional) {#transport-cert_path}

Specify path to Certificate file<br>

Default: -

### private_key_path (string, optional) {#transport-private_key_path}

Specify path to private Key file<br>

Default: -

### private_key_passphrase (string, optional) {#transport-private_key_passphrase}

public CA private key passphrase contained path<br>

Default: -

### client_cert_auth (bool, optional) {#transport-client_cert_auth}

When this is set Fluentd will check all incoming HTTPS requests<br>for a client certificate signed by the trusted CA, requests that<br>don't supply a valid client certificate will fail.<br>

Default: -

### ca_cert_path (string, optional) {#transport-ca_cert_path}

Specify private CA contained path<br>

Default: -

### ca_private_key_path (string, optional) {#transport-ca_private_key_path}

private CA private key contained path<br>

Default: -

### ca_private_key_passphrase (string, optional) {#transport-ca_private_key_passphrase}

private CA private key passphrase contained path<br>

Default: -


