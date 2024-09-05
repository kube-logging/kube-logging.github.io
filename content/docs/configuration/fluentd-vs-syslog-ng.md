---
title: Which log forwarder to use
linktitle: Fluentd or syslog-ng
weight: 10
---

The Logging operator supports [Fluentd](https://www.fluentd.org/) and syslog-ng (via the [AxoSyslog syslog-ng distribution](https://axoflow.com/docs/axosyslog-core/)) as log forwarders. The log forwarder instance receives, filters, and transforms the incoming logs, and transfers them to one or more destination outputs. Which one to use depends on your logging requirements.

 <!-- (note that you can use both syslog-ng and Fluentd side-by-side, but in this case you have to explicitly configure your Fluent Bit instances to FIXME) -->

The following points help you decide which forwarder to use.

- The forwarders support different outputs. If the output you want to use is supported by only one forwarder, use that.
- If the volume of incoming log messages is high, use syslog-ng, as its multithreaded processing provides higher performance.
- If you have many logging flows or need complex routing or log message processing, use syslog-ng.

> Note: Depending on the log forwarder you choose, the CRDs you need to create and configure will differ.

{{< include-headless "syslog-ng-minimum-version.md" >}}
