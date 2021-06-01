---
title: Common
weight: 200
generated_file: true
---

## ImageSpec

ImageSpec struct hold information about image specification

### repository (string, optional) {#imagespec-repository}

Default: -

### tag (string, optional) {#imagespec-tag}

Default: -

### pullPolicy (string, optional) {#imagespec-pullpolicy}

Default: -

### imagePullSecrets ([]corev1.LocalObjectReference, optional) {#imagespec-imagepullsecrets}

Default: -


## Metrics

Metrics defines the service monitor endpoints

### interval (string, optional) {#metrics-interval}

Default: -

### timeout (string, optional) {#metrics-timeout}

Default: -

### port (int32, optional) {#metrics-port}

Default: -

### path (string, optional) {#metrics-path}

Default: -

### serviceMonitor (bool, optional) {#metrics-servicemonitor}

Default: -

### serviceMonitorConfig (ServiceMonitorConfig, optional) {#metrics-servicemonitorconfig}

Default: -

### prometheusAnnotations (bool, optional) {#metrics-prometheusannotations}

Default: -


## ServiceMonitorConfig

ServiceMonitorConfig defines the ServiceMonitor properties

### additionalLabels (map[string]string, optional) {#servicemonitorconfig-additionallabels}

Default: -

### honorLabels (bool, optional) {#servicemonitorconfig-honorlabels}

Default: -

### relabelings ([]*v1.RelabelConfig, optional) {#servicemonitorconfig-relabelings}

Default: -

### metricRelabelings ([]*v1.RelabelConfig, optional) {#servicemonitorconfig-metricrelabelings}

Default: -


## Security

Security defines Fluentd, Fluentbit deployment security properties

### serviceAccount (string, optional) {#security-serviceaccount}

Default: -

### roleBasedAccessControlCreate (*bool, optional) {#security-rolebasedaccesscontrolcreate}

Default: -

### podSecurityPolicyCreate (bool, optional) {#security-podsecuritypolicycreate}

Default: -

### securityContext (*corev1.SecurityContext, optional) {#security-securitycontext}

Default: -

### podSecurityContext (*corev1.PodSecurityContext, optional) {#security-podsecuritycontext}

Default: -


