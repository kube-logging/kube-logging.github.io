---
title: Parsing custom date formats
weight: 80
---

By default, the syslog-ng aggregator uses the time when a message has been received on its input source as the timestamp. If you want to use the timestamp written in the message metadata, you can use a [date-parser](https://axoflow.com/docs/axosyslog-core/chapter-parsers/date-parser/date-parser-options/).

Available in Logging operator version 4.5 and later.

To use the timestamps written by the container runtime (_cri_ or _docker_) and parsed by Fluent Bit, define the `sourceDateParser` in the _syslog-ng_ spec.

```yaml
kind: Logging
metadata:
  name: example
spec:
  syslogNG:
    sourceDateParser: {}
```

You can also define your own parser format and template. The following example shows the default values.

```yaml
kind: Logging
metadata:
  name: example
spec:
  syslogNG:
    sourceDateParser:
      format: "%FT%T.%f%z"
      template: "${json.time}"
```
