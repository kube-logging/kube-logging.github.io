---
title: Rewrite
weight: 200
---

Rewrite filters can be used to modify record contents. Logging operator currently supports the following rewrite functions:

- [rename](#rename)
- [set](#set)
- [substitute](#subst)
- [unset](#unset)

> Note: All rewrite functions support an optional `condition` which has the same syntax as the [match filter](../match/).

## Rename

The `rename` function changes the name of an existing field name.

```yaml
  filters:
  - rewrite:
    - rename:
        oldName: "json.kubernetes.labels.app"
        newName: "json.kubernetes.labels.app.kubernetes.io/name"
```

## Set

The `set` function sets the value of a field.

```yaml
  filters:
  - rewrite:
    - set:
        field: "json.kubernetes.cluster"
        value: "prod-us"
```

## Substitute (subst) {#subst}

The `subst` function replaces parts of a field with a replacement value based on a pattern.

```yaml
  filters:
  - rewrite:
    - subst:
        pattern: "\d\d\d\d-\d\d\d\d-\d\d\d\d-\d\d\d\d"
        replace: "[redacted bank card number]"
        field: "MESSAGE"
```

The function also supports the `type` and `flags` fields for specifying pattern type and flags as described in the [match expression regexp function](../match/).

## Unset

You can unset macros or fields of the message.

> Note: Unsetting a field completely deletes any previous value of the field.

```yaml
  filters:
  - rewrite:
    - unset:
        field: "json.kubernetes.cluster"
```
