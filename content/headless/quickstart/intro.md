
This guide shows you how to collect application and container logs in Kubernetes using the Logging operator.

The Logging Operator itself doesn't store any logs, for demonstration purposes a special workload will be
deployed to the cluster to let you observe the logs flowing through the system.

The Logging operator collects all logs from the cluster, selects the specific logs based on pod labels and routes those to the defined output.
For more details about the Logging operator, see the [Logging operator overview]({{< relref "/docs/_index.md">}}).

> Note: this example aims to be simple enough to understand the basic capabilities of the operator. For a production ready setup, please take a look at {{% xref "/docs/logging-infrastructure/" %}} and {{% xref "/docs/operation/" %}}
