---
title: Routing your logs with syslog-ng
shorttitle: Log routing with syslog-ng
weight: 200
---

> Note: This page describes routing logs with **syslog-ng**. If you are using Fluentd to route your log messages, see {{% xref "/docs/one-eye/logging-operator/configuration/log-routing.md" %}}.

{{< include-headless "syslog-ng-minimum-version.md" "one-eye/logging-operator" >}}

The first step to process your logs is to select which logs go where.

The `match` field of the `SyslogNGFlow` and `SyslogNGClusterFlow` resources define the routing rules of the logs.

> Note: Fluentd can use only metadata to route the logs. When using [syslog-ng filter expressions](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/65#TOPIC-1829159), you can filter both on metadata and log content as well.
>
> The syntax of syslog-ng match statement is slightly different from the Fluentd match statements.

Available routing metadata keys:

| Name | Type | Description | Empty |
|------|------|-------------|-------|
| namespaces | []string | List of matching namespaces | All namespaces |
| labels | map[string]string | Key - Value pairs of labels | All labels |
| hosts | []string | List of matching hosts | All hosts |
| container_names | []string | List of matching containers (not Pods) | All containers |

## Match statement

Match expressions are basically a combination of filtering functions using the `and`, `or`, and `not` boolean operators.
Currently, only a pattern matching function is supported (called [`match`](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/68#TOPIC-1829171) in syslog-ng parlance, but renamed to `regexp` in the CRD to avoid confusion).

The `match` field can have one of the following options:

```yaml
  match:
    and: <list of nested match expressions>  // Logical AND between expressions
    or: <list of nested match expressions>   // Logical OR between expressions
    not: <nested match expression>           // Logical NOT of an expression
    regexp: ... // Pattern matching on a field's value or a templated value
```

The `regexp` field (called [`match`](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/68#TOPIC-1829171) in syslog-ng parlance, but renamed to `regexp` in the CRD to avoid confusion)) can have the following fields:

```yaml
  regexp:
    pattern: <a pattern string>                            // Pattern match against, e.g. "my-app-\d+". The pattern's type is determined by the type field.
    value: <a field reference>                             // Reference to a field whose value to match. If this field is set, the template field cannot be used.
    template: <a templated string combining field values>  // Template expression whose value to match. If this field is set, the value field cannot be used. For more info, see https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/74#TOPIC-1829197
    type: <pattern type>                                   // Pattern type. Default is PCRE. For more info, see https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/81#TOPIC-1829223
    flags: <list of flags>                                 // Pattern flags. For more info, see https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/81#TOPIC-1829224
```

{{< warning >}}You need to use the `json.` prefix in field names.{{< /warning >}}

You can reference fields using the *dot notation*, for example, if the log contains `{"kubernetes": {"namespace_name": "default"}}`, then you can reference the `namespace_name` field using `json.kubernetes.namespace_name`.

The following example filters for specific Pod labels:

```yaml
  match:
    and:
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/instance
        pattern: one-eye-log-generator
        type: string
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/name
        pattern: log-generator
        type: string
```

<!-- FIXME adapt Fluentd examples/add syslog-ng specific ones -->
