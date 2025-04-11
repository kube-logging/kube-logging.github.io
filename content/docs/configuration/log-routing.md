---
title: Routing your logs with Fluentd match directives
linktitle: Log routing with Fluentd
weight: 200
aliases:
    - /docs/one-eye/logging-operator/log-routing/
---

> Note: This page describes routing logs with **Fluentd**. If you are using syslog-ng to route your log messages, see {{% xref "/docs/configuration/log-routing-syslog-ng.md" %}}.

The first step to process your logs is to select which logs go where.
The Logging operator uses Kubernetes labels, namespaces and other metadata
to separate different log flows.

Available routing metadata keys:

| Name | Type | Description | Empty |
|------|------|-------------|-------|
| namespaces | []string | List of matching namespaces | All namespaces |
| namespaces_regex | []string | List of matching ruby regex for namespaces | All namespaces |
| labels | map[string]string | Key - Value pairs of labels | All labels |
| hosts | []string | List of matching hosts | All hosts |
| container_names | []string | List of matching containers (not Pods) | All containers |

## Match statement

To select or exclude logs you can use the `match` statement. Match is a collection
of `select` and `exclude` expressions. In both expression you can use the `labels`
attribute to filter for pod's labels. Moreover, in Cluster flow you can use `namespaces` and `namespaces_regex`
as a selecting or excluding criteria.

If you specify more than one label in a `select` or `exclude` expression, the labels have a logical AND connection between them. For example, an `exclude` expression with two labels excludes messages that have both labels. If you want an OR connection between labels, list them in separate expressions. For example, to exclude messages that have one of two specified labels, create a separate `exclude` expression for each label.

The `select` and `exclude` statements are evaluated **in order**!

Without at least one `select` criteria, no messages will be selected!

Flow:

```yaml
  kind: Flow
  metadata:
    name: flow-sample
  spec:
    match:
      - exclude:
          labels:
            exclude-this: label
      - select:
          labels:
            app: nginx
            label/xxx: example
```

ClusterFlow:

```yaml
  kind: ClusterFlow
  metadata:
    name: flow-sample
  spec:
    match:
      - exclude:
          labels:
            exclude-this: label
          namespaces:
            - developer
      - select:
          labels:
            app: nginx
            label/xxx: example
          namespaces:
            - production
            - beta
```

## Examples

### Example 0. Select all logs

To select all logs, or if you only want to exclude some logs but retain others you need an empty select statement.

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Flow
  metadata:
    name: flow-all
    namespace: default
  spec:
    localOutputRefs:
      - forward-output-sample
    match:
      - select: {}
  ```

### Example 1. Select logs by label

Select logs with `app: nginx` labels from the namespace:

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Flow
  metadata:
    name: flow-sample
    namespace: default
  spec:
    localOutputRefs:
      - forward-output-sample
    match:
      - select:
          labels:
            app: nginx
  ```

### Example 2. Exclude logs by label

Exclude logs with `app: nginx` labels from the namespace

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Flow
  metadata:
    name: flow-sample
    namespace: default
  spec:
    localOutputRefs:
      - forward-output-sample
    match:
      - exclude:
          labels:
            app: nginx
      - select: {}
  ```

### Example 3. Exclude and select logs by label

Select logs with `app: nginx` labels from the `default` namespace but exclude logs with `env: dev` labels

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Flow
  metadata:
    name: flow-sample
    namespace: default
  spec:
    localOutputRefs:
      - forward-output-sample
    match:
      - exclude:
          labels:
            env: dev
      - select:
          labels:
            app: nginx
  ```

### Example 4. Exclude cluster logs by namespace

Select `app: nginx` from all namespaces except from `dev` and `sandbox`

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: ClusterFlow
  metadata:
    name: clusterflow-sample
  spec:
    globalOutputRefs:
      - forward-output-sample
    match:
      - exclude:
          namespaces:
            - dev
            - sandbox
      - select:
          labels:
            app: nginx
  ```

### Example 5. Exclude cluster logs by ruby regex namespace

Select `app: nginx` from all namespaces except from those ending with `-dev`

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: ClusterFlow
  metadata:
    name: clusterflow-sample
  spec:
    globalOutputRefs:
      - forward-output-sample
    match:
      - exclude:
          namespaces_regex:
            - .*-dev$
      - select:
          labels:
            app: nginx
  ```

### Example 5. Exclude and select cluster logs by namespace and ruby regex

Select `app: nginx` from `infra` namespace and all namespaces starting with `prod-` but exclude cluster logs from `sandbox` namespace and all namespaces starting with `dev-`,

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: ClusterFlow
  metadata:
    name: clusterflow-sample
  spec:
    globalOutputRefs:
      - forward-output-sample
    match:
      - exclude:
          namespaces:
            - sandbox
          namespaces_regex:
            - ^dev-.*
      - select:
          labels:
            app: nginx
          namespaces:
            - infra
          namespaces_regex:
            - ^prod-.*
  ```

### Example 6. Multiple labels - AND

Exclude logs that have both the `app: nginx` and `app.kubernetes.io/instance: nginx-demo` labels

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Flow
  metadata:
    name: flow-sample
    namespace: default
  spec:
    localOutputRefs:
      - forward-output-sample
    match:
      - exclude:
          labels:
            app: nginx
            app.kubernetes.io/instance: nginx-demo
      - select: {}
  ```

### Example 6. Multiple labels - OR

Exclude logs that have either the `app: nginx` or the `app.kubernetes.io/instance: nginx-demo` labels

  ```yaml
  apiVersion: logging.banzaicloud.io/v1beta1
  kind: Flow
  metadata:
    name: flow-sample
    namespace: default
  spec:
    localOutputRefs:
      - forward-output-sample
    match:
      - exclude:
          labels:
            app: nginx
      - exclude:
          labels:
            app.kubernetes.io/instance: nginx-demo
      - select: {}
  ```
