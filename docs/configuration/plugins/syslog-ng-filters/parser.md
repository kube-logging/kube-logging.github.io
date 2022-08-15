---
title: Parser
weight: 200
---

Parser filters can be used to extract key-value pairs from message data. Logging operator currently supports the following parsers:

- [regexp](#regexp)
- [syslog-parser](#syslog)

## Regexp parser {#regexp}

The regexp parser can use regular expressions to parse fields from a message.

```yaml
  filters:
  - parser:
      regexp:
        patterns:
        - ".*test_field -> (?<test_field>.*)$"
        prefix: .regexp.
```

For details, see the [syslog-ng documentation](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.36/administration-guide/90#TOPIC-1768848).

## Syslog parser {#syslog}

The syslog parser can parse syslog messages. For details, see the [syslog-ng documentation](https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/83#TOPIC-1829231).

```yaml
  filters:
  - parser:
      syslog-parser: {} 
```
