---
title: Extensions
weight: 1000
aliases:
    - /docs/one-eye/logging-operator/extensions/
---

Logging extensions were specifically developed to solve the problems of users:

- Collecting Kubernetes events to provide insight into what is happening
inside a cluster, such as decisions made by the scheduler, or
why some pods were evicted from the node.
- Collect logs from the nodes like `kubelet` logs.
- Collect logs from files on the nodes, for example, `audit` logs, or the `systemd` journal.

Logging-extensions are now part of Logging-operator by default.

## Features

{{< toc >}}

> Check our [configuration snippets](https://github.com/banzaicloud/logging-operator/tree/master/config/samples/extensions) for examples.
