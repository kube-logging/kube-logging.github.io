---
title: Optimization
weight: 1200
aliases:
    - /docs/logging-operator/optimization/
---

## Watch specific resources

The Logging operator watches resources in all namespaces, which is required because it manages cluster-scoped objects, and also objects in multiple namespaces.

However, in a large-scale infrastructure, where the number of resources is large, it makes sense to limit the scope of resources monitored by the Logging operator to save considerable amount of memory and container restarts.

Starting with Logging operator version 3.12.0, this is now available by passing the following command-line arguments to the operator.

- `watch-namespace`: Watch only objects in this namespace. Note that even if the `watch-namespace` option is set, the operator must watch certain objects (like `Flows` and `Outputs`) in every namespace.
- `watch-logging-name`: Logging resource name to optionally filter the list of watched objects based on which logging they belong to by checking the `app.kubernetes.io/managed-by` label.
