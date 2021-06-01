---
title: SumoLogic
weight: 200
generated_file: true
---

# Sumo Logic collection solution for Kubernetes
## Overview
More info at https://github.com/SumoLogic/sumologic-kubernetes-collection

## Configuration
## SumoLogic

### source_category (string, optional) {#sumologic-source_category}

Source Category <br>

Default:  "%{namespace}/%{pod_name}"

### source_category_replace_dash (string, optional) {#sumologic-source_category_replace_dash}

Source Category Replace Dash <br>

Default:  "/"

### source_category_prefix (string, optional) {#sumologic-source_category_prefix}

Source Category Prefix <br>

Default:  kubernetes/

### source_name (string, optional) {#sumologic-source_name}

Source Name <br>

Default:  "%{namespace}.%{pod}.%{container}"

### log_format (string, optional) {#sumologic-log_format}

Log Format <br>

Default:  json

### source_host (string, optional) {#sumologic-source_host}

Source Host <br>

Default:  ""

### exclude_container_regex (string, optional) {#sumologic-exclude_container_regex}

Exclude Container Regex <br>

Default:  ""

### exclude_facility_regex (string, optional) {#sumologic-exclude_facility_regex}

Exclude Facility Regex <br>

Default:  ""

### exclude_host_regex (string, optional) {#sumologic-exclude_host_regex}

Exclude Host Regex <br>

Default:  ""

### exclude_namespace_regex (string, optional) {#sumologic-exclude_namespace_regex}

Exclude Namespace Regex <br>

Default:  ""

### exclude_pod_regex (string, optional) {#sumologic-exclude_pod_regex}

Exclude Pod Regex <br>

Default:  ""

### exclude_priority_regex (string, optional) {#sumologic-exclude_priority_regex}

Exclude Priority Regex <br>

Default:  ""

### exclude_unit_regex (string, optional) {#sumologic-exclude_unit_regex}

Exclude Unit Regex <br>

Default:  ""

### tracing_format (*bool, optional) {#sumologic-tracing_format}

Tracing Format <br>

Default:  false

### tracing_namespace (string, optional) {#sumologic-tracing_namespace}

Tracing Namespace <br>

Default:  "namespace"

### tracing_pod (string, optional) {#sumologic-tracing_pod}

Tracing Pod <br>

Default:  "pod"

### tracing_pod_id (string, optional) {#sumologic-tracing_pod_id}

Tracing Pod ID <br>

Default:  "pod_id"

### tracing_container_name (string, optional) {#sumologic-tracing_container_name}

Tracing Container Name <br>

Default:  "container_name"

### tracing_host (string, optional) {#sumologic-tracing_host}

Tracing Host <br>

Default:  "hostname"

### tracing_label_prefix (string, optional) {#sumologic-tracing_label_prefix}

Tracing Label Prefix <br>

Default:  "pod_label_"

### tracing_annotation_prefix (string, optional) {#sumologic-tracing_annotation_prefix}

Tracing Annotation Prefix <br>

Default:  "pod_annotation_"

### source_host_key_name (string, optional) {#sumologic-source_host_key_name}

Source HostKey Name <br>

Default:  "_sourceHost"

### source_category_key_name (string, optional) {#sumologic-source_category_key_name}

Source CategoryKey Name <br>

Default:  "_sourceCategory"

### source_name_key_name (string, optional) {#sumologic-source_name_key_name}

Source NameKey Name <br>

Default:  "_sourceName"

### collector_key_name (string, optional) {#sumologic-collector_key_name}

CollectorKey Name <br>

Default:  "_collector"

### collector_value (string, optional) {#sumologic-collector_value}

Collector Value <br>

Default:  "undefined"


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
