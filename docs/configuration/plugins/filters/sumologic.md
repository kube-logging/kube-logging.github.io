---
title: SumoLogic
weight: 200
generated_file: true
---

# Sumo Logic collection solution for Kubernetes
## Overview
More info at https://github.com/SumoLogic/sumologic-kubernetes-collection

## Configuration
### SumoLogic
| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| source_category | string | No |  "%{namespace}/%{pod_name}" | Source Category <br> |
| source_category_replace_dash | string | No |  "/" | Source Category Replace Dash <br> |
| source_category_prefix | string | No |  kubernetes/ | Source Category Prefix <br> |
| source_name | string | No |  "%{namespace}.%{pod}.%{container}" | Source Name <br> |
| log_format | string | No |  json | Log Format <br> |
| source_host | string | No |  "" | Source Host <br> |
| exclude_container_regex | string | No |  "" | Exclude Container Regex <br> |
| exclude_facility_regex | string | No |  "" | Exclude Facility Regex <br> |
| exclude_host_regex | string | No |  "" | Exclude Host Regex <br> |
| exclude_namespace_regex | string | No |  "" | Exclude Namespace Regex <br> |
| exclude_pod_regex | string | No |  "" | Exclude Pod Regex <br> |
| exclude_priority_regex | string | No |  "" | Exclude Priority Regex <br> |
| exclude_unit_regex | string | No |  "" | Exclude Unit Regex <br> |
| tracing_format | *bool | No |  false | Tracing Format <br> |
| tracing_namespace | string | No |  "namespace" | Tracing Namespace <br> |
| tracing_pod | string | No |  "pod" | Tracing Pod <br> |
| tracing_pod_id | string | No |  "pod_id" | Tracing Pod ID <br> |
| tracing_container_name | string | No |  "container_name" | Tracing Container Name <br> |
| tracing_host | string | No |  "hostname" | Tracing Host <br> |
| tracing_label_prefix | string | No |  "pod_label_" | Tracing Label Prefix <br> |
| tracing_annotation_prefix | string | No |  "pod_annotation_" | Tracing Annotation Prefix <br> |
| source_host_key_name | string | No |  "_sourceHost" | Source HostKey Name <br> |
| source_category_key_name | string | No |  "_sourceCategory" | Source CategoryKey Name <br> |
| source_name_key_name | string | No |  "_sourceName" | Source NameKey Name <br> |
| collector_key_name | string | No |  "_collector" | CollectorKey Name <br> |
| collector_value | string | No |  "undefined" | Collector Value <br> |
 #### Example `Parser` filter configurations
 ```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: demo-flow
spec:
  filters:
    - sumologic:
        source_name: "elso"
  selectors: {}
  localOutputRefs:
    - demo-output
 ```

 #### Fluentd Config Result
 ```yaml
<filter **>
  @type kubernetes_sumologic
  @id test_sumologic
  source_name elso
</filter>
 ```

---
