---
title: Multiple Fluent Bits in the cluster
weight: 500
---

In case there are different node groups in a cluster, you might need different Fluent Bit configurations on the different node groups. Starting with Logging operator version 4.2, you can do that by using the FluentbitAgent CRD. This also allows you to create hard multitenancy on the node group level.

For details on using the FluentbitAgent CRD, see {{% xref "/docs/logging-infrastructure/fluentbit.md" %}}.

To configure multiple FluentbitAgent CRDs for a cluster, complete the following steps. The examples refer to a scenario where you have two node groups that have the Kubernetes label `nodeGroup=A` and `nodeGroup=B`.

1. If you are updating an existing deployment, make sure that it already uses a Logging configuration based on FluentbitAgent CRD. If not, first [migrate your configuration to use a FluentbitAgent CRD]({{< relref "/docs/logging-infrastructure/fluentbit.md#migrating" >}}).
1. Edit your existing FluentbitAgent CRD, and set the **spec.nodeSelector** field so it applies only to the node group you want to apply this Fluent Bit configuration on, for example, nodes that have the label `nodeGroup=A`. For details, see [nodeSelector in the Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector).

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: FluentbitAgent
    metadata:
      # Use the same name as the logging resource does
      name: multi
    spec:
      positiondb:
        hostPath:
          path: ""
      bufferStorageVolume:
        hostPath:
          path: ""
      nodeSelector:
        nodeGroup: "A"
    ```

    > Note: If your Logging resource has its **spec.loggingRef** parameter set, set the same value in the **spec.loggingRef** parameter of the FluentbitAgent resource.

    Set other [FluentbitAgent parameters]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md" >}}) as needed for your environment.

1. Create a new FluentbitAgent CRD, and set the **spec.nodeSelector** field so it applies only to the node group you want to apply this Fluent Bit configuration on, for example, nodes that have the label `nodeGroup=B`. For details, see [nodeSelector in the Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector).

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: FluentbitAgent
    metadata:
      name: multi-B
    spec:
      nodeSelector:
        nodeGroup: "B"
    ```

    > Note: If your Logging resource has its **spec.loggingRef** parameter set, set the same value in the **spec.loggingRef** parameter of the FluentbitAgent resource.

    Set other [FluentbitAgent parameters]({{< relref "/docs/configuration/crds/v1beta1/fluentbit_types.md" >}}) as needed for your environment.

1. Create the Flow resources to route the log messages to the outputs. For example, you can select and exclude logs based on their node group labels.

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Flow
    metadata:
      name: "flow-for-nodegroup-A"
    spec:
      match:
        - select:
          labels:
            nodeGroup: "A"
      localOutputRefs:
        - "output-for-nodegroup-A"
    ```

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Flow
    metadata:
      name: "flow-for-nodegroup-B"
    spec:
      match:
        - select:
          labels:
            nodeGroup: "B"
      localOutputRefs:
        - "output-for-nodegroup-B"
    ```

    > Note: If your Logging resource has its **spec.loggingRef** parameter set, set the same value in the **spec.loggingRef** parameter of the Flow resource.

    Set other [Flow parameters]({{< relref "/docs/configuration/crds/v1beta1/flow_types.md" >}}) as needed for your environment.

1. Create the outputs (called `"output-for-nodegroup-A"` and `"output-for-nodegroup-B"`) for the Flows.
