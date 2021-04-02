---
title: LoggingSpec
weight: 200
generated_file: true
---

### LoggingSpec
#### LoggingSpec defines the desired state of Logging

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| loggingRef | string | No | - | Reference to the logging system. Each of the `loggingRef`s can manage a fluentbit daemonset and a fluentd statefulset.<br> |
| flowConfigCheckDisabled | bool | No | - | Disable configuration check before applying new fluentd configuration.<br> |
| flowConfigOverride | string | No | - | Override generated config. This is a *raw* configuration string for troubleshooting purposes.<br> |
| fluentbit | *FluentbitSpec | No | - | Fluentbit daemonset configuration.<br> |
| fluentd | *FluentdSpec | No | - | Fluentd statefulset configuration<br> |
| defaultFlow | *DefaultFlowSpec | No | - | Default flow for unmatched logs. This Flow configuration collects all logs that didn't matched any other Flow.<br> |
| globalFilters | []Filter | No | - | Global filters to apply on logs before any match or filter mechanism.<br> |
| watchNamespaces | []string | No | - | Limit namespaces to watch Flow and Output custom reasources.<br> |
| controlNamespace | string | Yes | - | Namespace for cluster wide configuration resources like CLusterFlow and ClusterOutput.<br>This should be a protected namespace from regular users.<br>Resources like fluentbit and fluentd will run in this namespace as well.<br> |
| allowClusterResourcesFromAllNamespaces | bool | No | - | Allow configuration of cluster resources from any namespace. Mutually exclusive with ControlNamespace restriction of Cluster resources<br> |
| nodeAgents | []*NodeAgent | No | - | NodeAgent Configuration<br> |
| enableRecreateWorkloadOnImmutableFieldChange | bool | No | - | EnableRecreateWorkloadOnImmutableFieldChange enables the operator to recreate the<br>fluentbit daemonset and the fluentd statefulset (and possibly other resource in the future)<br>in case there is a change in an immutable field<br>that otherwise couldn't be managed with a simple update.<br> |
### LoggingStatus
#### LoggingStatus defines the observed state of Logging

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| configCheckResults | map[string]bool | No | - |  |
### Logging
#### Logging is the Schema for the loggings API

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
|  | metav1.TypeMeta | Yes | - |  |
| metadata | metav1.ObjectMeta | No | - |  |
| spec | LoggingSpec | No | - |  |
| status | LoggingStatus | No | - |  |
### LoggingList
#### LoggingList contains a list of Logging

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
|  | metav1.TypeMeta | Yes | - |  |
| metadata | metav1.ListMeta | No | - |  |
| items | []Logging | Yes | - |  |
### DefaultFlowSpec
#### DefaultFlowSpec is a Flow for logs that did not match any other Flow

| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| filters | []Filter | No | - |  |
| outputRefs | []string | No | - | Deprecated<br> |
| globalOutputRefs | []string | No | - |  |
