---
title: Routing your logs with syslog-ng
linktitle: Log routing with syslog-ng
weight: 200
---

> Note: This page describes routing logs with **syslog-ng**. If you are using Fluentd to route your log messages, see {{% xref "/docs/configuration/log-routing.md" %}}.

{{< include-headless "syslog-ng-minimum-version.md" >}}

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

Match expressions select messages by applying patterns on the content or metadata of the messages. You can use simple string matching, and also complex regular expressions. You can combine matches using the `and`, `or`, and `not` boolean operators to create complex expressions to select or exclude messages as needed for your use case.

Currently, only a pattern matching function is supported (called [`match`](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/68#TOPIC-1829171) in syslog-ng parlance, but renamed to `regexp` in the CRD to avoid confusion).

The `match` field can have one of the following options:

- `regexp`: A pattern that matches the value of a field or a templated value. For example:

    ```yaml
      match:
        regexp: <parameters>
    ```

- `and`: Combines the nested match expressions with the logical AND operator.

    ```yaml
      match:
        and: <list of nested match expressions>
    ```

- `or`: Combines the nested match expressions with the logical OR operator.

    ```yaml
      match:
        or: <list of nested match expressions>
    ```

- `not`: Matches the logical NOT of the nested match expressions with the logical AND operator.

    ```yaml
      match:
        not: <list of nested match expressions>
    ```

## `regexp` patterns

The `regexp` field (called [`match`](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/68#TOPIC-1829171) in syslog-ng parlance, but renamed to `regexp` in the CRD to avoid confusion) defines the pattern that selects the matching messages. You can do two different kinds of matching:

- Find a pattern in the value of a field of the messages, for example, to select the messages of a specific application. To do that, set the `pattern` and `value` fields (and optionally the `type` and `flags` fields).
- Find a pattern in a template expression created from multiple fields of the message. To do that, set the `pattern` and `template` fields (and optionally the `type` and `flags` fields).

{{< include-headless "field-names-json-prefix.md" >}}

The following example filters for specific Pod labels:

```yaml
  match:
    and:
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/instance
        pattern: log-generator
        type: string
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/name
        pattern: log-generator
        type: string
```

## `regexp` parameters

The `regexp` field can have the following parameters:

### `pattern` (string) {#regexp-pattern class="property-required"}

Defines the pattern to match against the messages. The [`type` field](#regexp-type) determines how the pattern is interpreted (for example, string or regular expression).

### `value` (string) {#regexp-value class="property-optional"}

References a field of the message. The `pattern` is applied to the value of this field. If the `value` field is set, you cannot use the `template` field.

{{< include-headless "field-names-json-prefix.md" >}}

For example:

```yaml
  match:
    regexp:
      value: json.kubernetes.labels.app.kubernetes.io/name
      pattern: nginx
```

### `template` (string) {#regexp-template class="property-optional"}

Specifies a template expression that combines fields. The `pattern` is matched against the value of these combined fields. If the `template` field is set, you cannot use the `value` field. For details on template expressions, see the [syslog-ng documentation](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/74#TOPIC-1829197).

### `type` (string) {#regexp-type class="property-optional"}

Specifies how the `pattern` is interpreted. For details, see [Types of `regexp`](#regexp-types).

### `flags` (list) {#regexp-flags class="property-optional"}

Specifies flags for the `type` field.

## `regexp` types {#regexp-types}

By default, syslog-ng uses PCRE-style regular expressions. Since evaluating complex regular expressions can greatly increase CPU usage and are not always needed, you can following expression types:

- [Perl Compatible Regular Expressions (PCRE)](#pcre)
- [Literal string searches](#string)
- [Glob patterns](#glob) (without regular expression support)

### `pcre`

Description: Use Perl Compatible Regular Expressions (PCRE). If the type() parameter is not specified, syslog-ng uses PCRE regular expressions by default.

### `pcre` flags

PCRE regular expressions have the following flag options:

- `disable-jit`: Disable the [just-in-time compilation function](https://www.pcre.org/current/doc/html/pcre2jit.html) for PCRE regular expressions.
- `dupnames`: Allow using [duplicate names for named subpatterns](https://www.pcre.org/original/doc/html/pcrepattern.html#SEC16).
- `global`: Usable only in rewrite rules: match for every occurrence of the expression, not only the first one.
- `ignore-case`: Disable case-sensitivity.
- `newline`: When configured, it changes the newline definition used in PCRE regular expressions to accept either of the following:

    - a single carriage-return
    - linefeed
    - the sequence carriage-return and linefeed (\r, \n and \r\n, respectively)

    This newline definition is used when the circumflex and dollar patterns (^ and $) are matched against an input. By default, PCRE interprets the linefeed character as indicating the end of a line. It does not affect the \r, \n or \R characters used in patterns.
- `store-matches`: Store the matches of the regular expression into the $0, ... $255 variables. The $0 stores the entire match, $1 is the first group of the match (parentheses), and so on. Named matches (also called named subpatterns), for example (`?<name>...`), are stored as well. Matches from the last filter expression can be referenced in regular expressions.
- `unicode`: Use Unicode support for UTF-8 matches. UTF-8 character sequences are handled as single characters.
- `utf8`: An alias for the unicode flag.

For example:

```yaml
  match:
    and:
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/instance
        pattern: log-generator
        flag: ignore-case
```

### `string`

Description: Match the strings literally, without regular expression support. By default, only identical strings are matched. For partial matches, use the `flags: prefix` or `flags: substring` flags. For example, if the consider the following patterns.

```yaml
  match:
    and:
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/name
        pattern: log-generator
        type: string
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/name
        pattern: log-generator
        type: string
        flag: prefix
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/name
        pattern: log-generator
        type: string
        flag: substring
```

- The first matches only the `log-generator` label.
- The second matches labels beginning with `log-generator`, for example, `log-generator-1`.
- The third one matches labels that contain the `log-generator` string, for example, `my-log-generator`.

### `string` flags

Literal string searches have the following flags() options:

- `global`: Usable only in rewrite rules, match for every occurrence of the expression, not only the first one.
- `ignore-case`: Disables case-sensitivity.
- `prefix`: During the matching process, patterns (also called search expressions) are matched against the input string starting from the beginning of the input string, and the input string is matched only for the maximum character length of the pattern. The initial characters of the pattern and the input string must be identical in the exact same order, and the pattern's length is definitive for the matching process (that is, if the pattern is longer than the input string, the match will fail).

    For example, for the input string `exam`:

    - the following patterns will match:
        - `ex` (the pattern contains the initial characters of the input string in the exact same order)
        - `exam` (the pattern is an exact match for the input string)
    - the following patterns will not match:
        - `example` (the pattern is longer than the input string)
        - `hexameter` (the pattern's initial characters do not match the input string's characters in the exact same order, and the pattern is longer than the input string)

- `store-matches`: Stores the matches of the regular expression into the $0, ... $255 variables. The $0 stores the entire match, $1 is the first group of the match (parentheses), and so on. Named matches (also called named subpatterns), for example, (`?<name>...`), are stored as well. Matches from the last filter expression can be referenced in regular expressions.

    > NOTE: To convert match variables into a syslog-ng list, use the $* macro, which can be further manipulated using [List manipulation](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/76#TOPIC-1829203), or turned into a list in type-aware destinations.

- `substring`: The given literal string will match when the pattern is found within the input. Unlike `flags: prefix`, the pattern does not have to be identical with the given literal string.

### `glob`

Description: Match the strings against a pattern containing '*' and '?' wildcards, without regular expression and character range support. The advantage of glob patterns to regular expressions is that globs can be processed much faster.

- `*`: matches an arbitrary string, including an empty string
- `?`: matches an arbitrary character

> NOTE:
>
> - The wildcards can match the `/` character.
> - You cannot use the `*` and `?` characters literally in the pattern.

Glob patterns cannot have any flags.

## Examples

### Select all logs

To select all logs, or if you only want to exclude some logs but retain others you need an empty select statement.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: flow-all
  namespace: default
spec:
  match:
    regexp:
      value: json.kubernetes.labels.app.kubernetes.io/instance
      pattern: "*"
      type: glob
  localOutputRefs:
    - syslog-output
```

### Select logs by label

Select logs with `app: nginx` labels from the namespace:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: flow-app-nginx
  namespace: default
spec:
  match:
    regexp:
      value: json.kubernetes.labels.app.kubernetes.io/name
      pattern: nginx
      type: glob
  localOutputRefs:
    - syslog-output
```

### Exclude logs by label

Exclude logs with `app: nginx` labels from the namespace.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: flow-not-nginx
  namespace: default
spec:
  match:
    not:
      regexp:
        value: json.kubernetes.labels.app.kubernetes.io/name
        pattern: nginx
        type: glob
  localOutputRefs:
    - syslog-output
```

### Exclude and select logs by label

Exclude logs with `env: dev` labels but select `app: nginx` labels from the namespace.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: flow-not-nginx
  namespace: default
spec:
  match:
    and:
    - regexp:
        value: json.kubernetes.labels.app.kubernetes.io/name
        pattern: nginx
        type: glob
    - not:
        regexp:
          value: json.kubernetes.labels.app.kubernetes.io/env
          pattern: dev
          type: glob
  localOutputRefs:
    - syslog-output
```

<!-- FIXME how can you filter on namespaces with syslog-ng? -->
<!--### Exclude cluster logs by namespace

Exclude cluster logs from the `dev`, `sandbox` namespaces and select `app: nginx` from all namespaces

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGClusterFlow
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

### Exclude and select cluster logs by namespace

Exclude cluster logs from  `dev`, `sandbox` namespaces and select `app: nginx` from all `prod` and `infra` namespaces

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
          namespaces:
            - prod
            - infra
  ```
-->

### Multiple labels - AND

Exclude logs that have both the `app: nginx` and `app.kubernetes.io/instance: nginx-demo` labels.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: flow-sample
  namespace: default
spec:
  localOutputRefs:
    - forward-output-sample
  match:
    not:
      and:
      - regexp:
          value: json.kubernetes.labels.app.kubernetes.io/name
          pattern: nginx
          type: glob
      - regexp:
          value: json.kubernetes.labels.app.kubernetes.io/instance
          pattern: nginx-demo
          type: glob
```

### Multiple labels - OR

Exclude logs that have either the `app: nginx` or the `app.kubernetes.io/instance: nginx-demo` labels

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: SyslogNGFlow
metadata:
  name: flow-sample
  namespace: default
spec:
  localOutputRefs:
    - forward-output-sample
  match:
    not:
      or:
      - regexp:
          value: json.kubernetes.labels.app.kubernetes.io/name
          pattern: nginx
          type: glob
      - regexp:
          value: json.kubernetes.labels.app.kubernetes.io/instance
          pattern: nginx-demo
          type: glob
```

<!-- FIXME add content-filtering examples -->
