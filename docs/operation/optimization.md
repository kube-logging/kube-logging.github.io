---
title: Optimization
weight: 1200
aliases:
    - /docs/logging-operator/optimization/
---

## Watch specific resources

The logging-operator watches resources in all namespaces, which is required because it manages cluster scoped objects and objects in multiple namespaces as well.

However in a large-scale infrastructure, where the number of resources is large it makes sense to limit the scope of resources monitored by the logging-operator to save considerable amount of memory and container restarts.

Previously this wasn't possible, but as of logging-operator version 3.12.0 this is now available using command line arguments passed to the operator.

You can use the following logging-operator command line parameters:

- `watch-namespace` Namespace to filter the list of watched objects. Doesn't apply to objects where it is till required to watch in all namespaces, like `Flows` and `Outputs`.
- `watch-logging-name` Logging resource name to optionally filter the list of watched objects based on which logging they belong to by checking the `app.kubernetes.io/managed-by` label.


### Configure by One-eye operator's Observer CRD

```yalm
apiVersion: one-eye.banzaicloud.io/v1alpha1
kind: Observer
metadata:
  name: one-eye
spec:
  logging:
    operator:
      watchLoggingName: 'one-eye'
      watchNamespace: 'default'
```
