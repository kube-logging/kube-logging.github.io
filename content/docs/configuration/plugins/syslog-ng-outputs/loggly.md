---
title: Loggly output
weight: 200
generated_file: true
---

## Overview

The `loggly()` destination sends log messages to the [Loggly](https://www.loggly.com/) Logging-as-a-Service provider. You can send log messages over TCP, or encrypted with TLS. For details on the available options of the output, see the [documentation of the AxoSyslog syslog-ng distribution](https://axoflow.com/docs/axosyslog-core/chapter-destinations/configuring-destinations-loggly/).

## Prerequisites

You need a Loggly account and your user token to use this output.

## Configuration

### host (string, optional) {#loggly-host}

Address of the destination host 

Default: -

### tag (string, optional) {#loggly-tag}

Event tag [more information](https://documentation.solarwinds.com/en/success_center/loggly/content/admin/tags.htm) 

Default: -

### token (*secret.Secret, required) {#loggly-token}

Your Customer Token that you received from Loggly. For details, see the [documentation of the AxoSyslog syslog-ng distribution](https://axoflow.com/docs/axosyslog-core/chapter-destinations/configuring-destinations-loggly/reference-destination-loggly/#loggly-option-token).

Default: -

###  (SyslogOutput, required) {#loggly-}

syslog output configuration 

Default: -


