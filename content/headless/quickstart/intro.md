
This guide shows you how to collect application and container logs in Kubernetes using the Logging operator.

The Logging operator itself doesn't store any logs. For demonstration purposes, it can deploy a special workload to the cluster to let you observe the logs flowing through the system.

The Logging operator collects all logs from the cluster, selects the specific logs based on pod labels, and sends the selected log messages to the output.
For more details about the Logging operator, see the [Logging operator overview]({{< relref "/docs/_index.md">}}).

> Note: This example aims to be simple enough to understand the basic capabilities of the operator. For a production ready setup, see {{% xref "/docs/logging-infrastructure/" %}} and {{% xref "/docs/operation/" %}}.
