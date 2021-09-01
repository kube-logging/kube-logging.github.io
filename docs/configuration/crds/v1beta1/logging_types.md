---
title: LoggingSpec
weight: 200
generated_file: true
---

## LoggingSpec

LoggingSpec defines the desired state of Logging

### loggingRef (string, optional) {#loggingspec-loggingref}

Reference to the logging system. Each of the `loggingRef`s can manage a fluentbit daemonset and a fluentd statefulset.<br>

Default: -

### flowConfigCheckDisabled (bool, optional) {#loggingspec-flowconfigcheckdisabled}

Disable configuration check before applying new fluentd configuration.<br>

Default: -

### flowConfigOverride (string, optional) {#loggingspec-flowconfigoverride}

Override generated config. This is a *raw* configuration string for troubleshooting purposes.<br>

Default: -

### fluentbit (*FluentbitSpec, optional) {#loggingspec-fluentbit}

Fluentbit daemonset configuration.<br>

Default: -

### fluentd (*FluentdSpec, optional) {#loggingspec-fluentd}

Fluentd statefulset configuration<br>

Default: -

### defaultFlow (*DefaultFlowSpec, optional) {#loggingspec-defaultflow}

Default flow for unmatched logs. This Flow configuration collects all logs that didn't matched any other Flow.<br>

Default: -

### errorOutputRef (string, optional) {#loggingspec-erroroutputref}

GlobalOutput name to flush ERROR events to<br>

Default: -

### globalFilters ([]Filter, optional) {#loggingspec-globalfilters}

Global filters to apply on logs before any match or filter mechanism.<br>

Default: -

### watchNamespaces ([]string, optional) {#loggingspec-watchnamespaces}

Limit namespaces to watch Flow and Output custom reasources.<br>

Default: -

### controlNamespace (string, required) {#loggingspec-controlnamespace}

Namespace for cluster wide configuration resources like CLusterFlow and ClusterOutput.<br>This should be a protected namespace from regular users.<br>Resources like fluentbit and fluentd will run in this namespace as well.<br>

Default: -

### allowClusterResourcesFromAllNamespaces (bool, optional) {#loggingspec-allowclusterresourcesfromallnamespaces}

Allow configuration of cluster resources from any namespace. Mutually exclusive with ControlNamespace restriction of Cluster resources<br>

Default: -

### nodeAgents ([]*NodeAgent, optional) {#loggingspec-nodeagents}

NodeAgent Configuration<br>

Default: -

### enableRecreateWorkloadOnImmutableFieldChange (bool, optional) {#loggingspec-enablerecreateworkloadonimmutablefieldchange}

EnableRecreateWorkloadOnImmutableFieldChange enables the operator to recreate the<br>fluentbit daemonset and the fluentd statefulset (and possibly other resource in the future)<br>in case there is a change in an immutable field<br>that otherwise couldn't be managed with a simple update.<br>

Default: -


## LoggingStatus

LoggingStatus defines the observed state of Logging

### configCheckResults (map[string]bool, optional) {#loggingstatus-configcheckresults}

Default: -


## Logging

Logging is the Schema for the loggings API

###  (metav1.TypeMeta, required) {#logging-}

Default: -

### metadata (metav1.ObjectMeta, optional) {#logging-metadata}

Default: -

### spec (LoggingSpec, optional) {#logging-spec}

Default: -

### status (LoggingStatus, optional) {#logging-status}

Default: -


## LoggingList

LoggingList contains a list of Logging

###  (metav1.TypeMeta, required) {#logginglist-}

Default: -

### metadata (metav1.ListMeta, optional) {#logginglist-metadata}

Default: -

### items ([]Logging, required) {#logginglist-items}

Default: -


## DefaultFlowSpec

DefaultFlowSpec is a Flow for logs that did not match any other Flow

### filters ([]Filter, optional) {#defaultflowspec-filters}

Default: -

### outputRefs ([]string, optional) {#defaultflowspec-outputrefs}

Deprecated<br>

Default: -

### globalOutputRefs ([]string, optional) {#defaultflowspec-globaloutputrefs}

Default: -


