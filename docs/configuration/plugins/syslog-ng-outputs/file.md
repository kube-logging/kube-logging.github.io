---
title: File
weight: 200
---

The `file` output stores log records in a plain text file.

```yaml
  spec:
    file:
      path: /mnt/archive/logs/${YEAR}/${MONTH}/${DAY}/app.log
      create_dirs: true
```

For details on the available options of the output, see the [syslog-ng documentation](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/36#TOPIC-1829044).
