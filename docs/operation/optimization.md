---
title: Optimization
weight: 1200
aliases:
    - /docs/logging-operator/optimization/
---

## Watch specific resources

In a large-scale infrastructure, there can be use-cases when it is required to limit the resources monitored by logging-operator and this way save cpu, memory resources can be saved.

Yop can use the following loggin-oprtator commandline parameters:

- `watch-namespace` Namespace to filter the list of watched objects
- `watch-logging-name` Logging resource name to optionally filter the list of watched objects based on which logging they belong to by checking the app.kubernetes.io/managed-by label


