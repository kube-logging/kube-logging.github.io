---
title: Enhance K8s Metadata
weight: 200
generated_file: true
---

# [Enhance K8s Metadata](https://github.com/SumoLogic/sumologic-kubernetes-collection/tree/main/fluent-plugin-enhance-k8s-metadata)
## Overview
 Fluentd Filter plugin to fetch several metadata for a Pod

## Configuration
### EnhanceK8s
| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| in_namespace_path | []string | No |  ['$.namespace'] | parameters for read/write record <br> |
| in_pod_path | []string | No |  ['$.pod','$.pod_name'] | <br> |
| data_type | string | No |  metrics | Sumologic data type <br> |
| kubernetes_url | string | No |  nil | Kubernetes API URL <br> |
| client_cert | secret.Secret | No |  nil | Kubernetes API Client certificate <br> |
| client_key | secret.Secret | No |  nil | // Kubernetes API Client certificate key <br> |
| ca_file | secret.Secret | No |  nil | Kubernetes API CA file <br> |
| secret_dir | string | No |  /var/run/secrets/kubernetes.io/serviceaccount | Service account directory <br> |
| bearer_token_file | string | No |  nil | Bearer token path <br> |
| verify_ssl | *bool | No |  true | Verify SSL <br> |
| core_api_versions | []string | No |  ['v1'] | Kubernetes core API version (for different Kubernetes versions) <br> |
| api_groups | []string | No |  ["apps/v1", "extensions/v1beta1"] | Kubernetes resources api groups <br> |
| ssl_partial_chain | *bool | No |  false | if `ca_file` is for an intermediate CA, or otherwise we do not have the<br>root CA and want to trust the intermediate CA certs we do have, set this<br>to `true` - this corresponds to the openssl s_client -partial_chain flag<br>and X509_V_FLAG_PARTIAL_CHAIN <br> |
| cache_size | int | No |  1000 | Cache size  <br> |
| cache_ttl | int | No |  60*60*2 | Cache TTL <br> |
| cache_refresh | int | No |  60*60 | Cache refresh <br> |
| cache_refresh_variation | int | No |  60*15 | Cache refresh variation <br> |
 #### Example `EnhanceK8s` filter configurations
 ```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: demo-flow
spec:
  filters:
    - enhanceK8s: {]
  selectors: {}
  localOutputRefs:
    - demo-output
 ```

 #### Fluentd Config Result
 ```yaml
<filter **>
  @type enhance_k8s_metadata
  @id test_enhanceK8s
</filter>
 ```

---
