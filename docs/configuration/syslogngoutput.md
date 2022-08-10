---
title: SyslogNGOutput and SyslogNGClusterOutput
weight: 50
---

The `SyslogNGOutput` resource defines an output where your SyslogNGFlows can send the log messages. The output is a `namespaced` resource which means only a `SyslogNGFlow` within the *same* namespace can access it. You can use `secrets` in these definitions, but they must also be in the same namespace.
Outputs are the final stage for a `logging flow`. You can define multiple `SyslogNGoutputs` and attach them to multiple `SyslogNGFlows`.

`SyslogNGClusterOutput` defines a SyslogNGOutput **without** namespace restrictions. It is only evaluated in the `controlNamespace` by default unless `allowClusterResourcesFromAllNamespaces` is set to true.

> Note: `SyslogNGFlow` can be connected to `SyslogNGOutput` and `SyslogNGClusterOutput`, but `SyslogNGClusterFlow` can be attached only to `SyslogNGClusterOutput`.

<!-- FIXME link to a page that shows the supported syslog-ng outputs
- For the details of the supported output plugins, see {{% xref "/docs/logging-operator/configuration/plugins/outputs/_index.md" %}}.-->
- For the details of `SyslogNGOutput` custom resource, see {{% xref "/docs/logging-operator/configuration/crds/v1beta1/syslogng_output_types.md" %}}.
- For the details of `SyslogNGClusterOutput` custom resource, see {{% xref "/docs/logging-operator/configuration/crds/v1beta1/syslogng_clusteroutput_types.md" %}}.

<!-- FIXME add an example output -->
