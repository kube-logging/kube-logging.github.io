---
title: Relabel
weight: 200
generated_file: true
---

Available in Logging Operator version 4.2 and later.

The relabel output uses the [relabel output plugin of Fluentd](https://docs.fluentd.org/output/relabel) to route events back to a specific Flow, where they can be processed again.

This is useful, for example, if the metadata of the message doesn’t contain the information needed for the routing decision, but the content of the message contains the required information. In this case, you can create a Flow that extracts the required information from the content of the message (for example, using the {{% xref "/docs/configuration/plugins/filters/grep.md" %}} or the {{% xref "/docs/configuration/plugins/filters/parser.md" %}} filter plugins), and uses the relabel output to send the message to a different Flow.

The value of the `label` parameter of the relabel output must be the same as the value of the `flowLabel` parameter of the Flow (or ClusterFlow) where you want to send the messages.

For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: ClusterOutput
metadata:
  name: final-relabel
spec:
  relabel:
    label: '@final-flow'
---
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: serviceFlow1
  namespace: namespace1
spec:
  filters: []
  globalOutputRefs:
  - final-relabel
  match:
  - select:
      labels:
        app: service1
---
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: serviceFlow2
  namespace: namespace2
spec:
  filters: []
  globalOutputRefs:
  - final-relabel
  match:
  - select:
      labels:
        app: service2
---
apiVersion: logging.banzaicloud.io/v1beta1
kind: ClusterFlow
metadata:
  name: final-flow
spec:
  flowLabel: '@final-flow'
  includeLabelInRouter: false
  filters: []
```

Using the relabel output also makes it possible to pass the messages emitted by the {{% xref "/docs/configuration/plugins/filters/concat.md" %}} plugin in case of a timeout. Set the `timeout_label` of the concat plugin to the flowLabel of the flow where you want to send the timeout messages.

## Output Config

### label (string, required) {#output config-label}

Specifies new label for events 

Default: -


