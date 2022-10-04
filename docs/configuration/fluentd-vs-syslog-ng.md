---
title: Which log forwarder to use
shorttitle: Fluentd or syslog-ng
weight: 10
---

The Logging operator supports Fluentd and syslog-ng as log forwarders. The log forwarder instance receives, filters, and transforms the incoming the logs, and transfers them to one or more destination outputs. Which one to use depends on your logging requirements.

 <!-- (note that you can use both syslog-ng and Fluentd side-by-side, but in this case you have to explicitly configure your Fluent Bit instances to FIXME) -->

The following points help you decide which forwarder to use.

- The forwarders support different outputs. If the output you want to use is supported only by one forwarder, use that.
- If the volume of incoming log messages is high, use syslog-ng, as its multithreaded processing provides higher performance.
- If you have lots of logging flows or need complex routing or log message processing, use syslog-ng.

> Note: Depending on which log forwarder you use, some of the CRDs you have to create and configure are different.

{{< include-headless "syslog-ng-minimum-version.md" "one-eye/logging-operator" >}}
