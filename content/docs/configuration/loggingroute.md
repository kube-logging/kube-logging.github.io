---
title: Logging route
weight: 35
---

> Note: The `LoggingRoute` resource is available in Logging operator 4.4 and later.

A `LoggingRoute` defines a global rule that instructs the `FluentbitAgent` resources of the same `Logging` resource to route logs to different target `Logging` aggregators (Fluentd or syslog-ng).

The routed logs are filtered based on the `watchNamespaces` and `watchNamespaceSelector` fields of the target `Logging` resources (which were originally used to limit which Flow and Output resources are processed by the `Logging` resource). This also means that the logs routed by ClusterFlows are limited to the above namespace list, as the aggregator doesn't receive any other logs.

For example, the following logging route configuration means that the `FluentbitAgent` resource in the `Logging` resource called `ops` routes logs to every aggregator whose `Logging` resource has the `tenant` label set. Each target aggregator receives only the logs of the namespaces it watches.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: LoggingRoute
metadata:
  name: tenants
spec:
  source: ops
  targets:
    matchExpressions:
    - key: tenant
      operator: Exists
```

## Status

The status of the `LoggingRoute` resource is populated with the targets and their namespaces. In case there is an issue, the `problems` field highlights issues that blocks a tenant from receiving any messages, while notices are only informational messages.

### Example with Logging resources and status

In this example, each tenant belongs to a development team (`team-a` and `team-b`), and each team has access only to its own logs. Every team has an `ops` and an `app` namespace, labeled with the team's tenant label:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  labels:
    tenant: team-a
  name: team-a
spec:
  controlNamespace: team-a-ops
  fluentd: {}
  watchNamespaceSelector:
    matchLabels:
      tenant: team-a
---
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  labels:
    tenant: team-b
  name: team-b
spec:
  controlNamespace: team-b-ops
  syslogNG: {}
  watchNamespaceSelector:
    matchLabels:
      tenant: team-b
```

> Note:
>
> - These logging resources don't have a corresponding `FluentbitAgent` resource defined as log collection will be handled by the `ops` tenant.
> - Using `loggingRef` is not required here.

The `ops` tenant where all logs should be available:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  labels:
    tenant: ops
  name: ops
spec:
  controlNamespace: ops
  fluentd: {}
  loggingRef: ops
---
apiVersion: logging.banzaicloud.io/v1beta1
kind: FluentbitAgent
metadata:
  name: ops
spec:
  loggingRef: ops
  forwardOptions:
    Workers: 0
  syslogng_output:
    Workers: 0
```

> Note:
>
> - `loggingRef`s are required here
> - `Workers: 0` is a workaround so that the processing of all tenants (outputs) isn't blocked if one or more tenant is unavailable

And finally the logging route with a populated status:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: LoggingRoute
metadata:
  name: tenants
spec:
  source: ops
  targets:
    matchExpressions:
    - key: tenant
      operator: Exists
status:
  notices:
  - tenant ops receives logs from ALL namespaces
  noticesCount: 1
  tenants:
  - name: team-a
    namespaces:
    - team-a-ops
    - team-a-app
  - name: team-b
    namespaces:
    - team-b-ops
    - team-b-app
  - name: ops
```

> Note: There is a notice that the ops tenant receives logs for all namespaces, which is exactly what we want here, but for a team or a customer level tenant it can easily be a misconfiguration issue, hence the notice.
