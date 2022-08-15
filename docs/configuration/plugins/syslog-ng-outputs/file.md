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
