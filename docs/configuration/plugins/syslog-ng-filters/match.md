---
title: Match
weight: 200
---

Match filters can be used to select the log records to process. These filters have the same options and syntax as the [syslog-ng flow match expressions]({{< relref "/docs/logging-operator/configuration/plugins/syslog-ng-filters/match.md" >}}).

```yaml
  filters:
  - match:
      or:
      - regexp:
          value: json.kubernetes.labels.app.kubernetes.io/name
          pattern: apache
          type: string
      - regexp:
          value: json.kubernetes.labels.app.kubernetes.io/name
          pattern: nginx
          type: string
```
