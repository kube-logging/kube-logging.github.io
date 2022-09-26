---
title: Kubernetes events, node logs, and logfiles
weight: 250
aliases:
    - /docs/one-eye/logging-operator/extensions/
---

The Logging extensions part of the Logging operator solves the following problems:

- Collect Kubernetes events to provide insight into what is happening inside a cluster, such as decisions made by the scheduler, or why some pods were evicted from the node.
- Collect logs from the nodes like `kubelet` logs.
- Collect logs from files on the nodes, for example, `audit` logs, or the `systemd` journal.
- Collect logs from legacy application log files.

Starting with Logging operator version 3.17.0, logging-extensions are open source and part of Logging operator.

## Features

Logging-operator handles the new features the well-known way: it uses custom resources to access the features. This way a simple `kubectl apply` with a particular parameter set initiates a new feature. Extensions supports three different custom resource types:

- Logging-operator handles the new features the well-known way: it uses custom resources to access the features. This way a simple `kubectl apply` with a particular parameter set initiates a new feature. Extensions supports three different custom resource types:

- [Event-tailer]({{< relref "kubernetes-event-tailer.md" >}}) listens for Kubernetes events and transmits their changes to stdout, so the Logging operator can process them.
- [Host-tailer]({{< relref "kubernetes-host-tailer.md" >}}) tails custom files and transmits their changes to stdout. This way the Logging operator can process them.
    Kubernetes host tailer allows you to tail logs like `kubelet`, `audit` logs, or the `systemd` journal from the nodes.
- [Tailer-webhook]({{< relref "tailer-webhook.md" >}}) is a different approach for the same problem: parsing legacy application's log file. Instead of running a `host-tailer` instance on every node, `tailer-webhook` attaches a `sidecar container` to the pod, and reads the specified file(s).

> Check our [configuration snippets](https://github.com/banzaicloud/logging-operator/tree/master/config/samples/extensions) for examples.
