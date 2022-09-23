---
title: Kubernetes Event Tailer
weight: 100
aliases:
    - /docs/logging-extensions/kubernetes-event-tailer/
---

Kubernetes events are objects that provide insight into what is happening inside a cluster, such as what decisions were made by the scheduler or why some pods were evicted from the node. Event tailer listens for Kubernetes events and transmits their changes to stdout, so the Logging operator can process them.

![Event tailer](../logging-extensions-event-tailer.png)

The operator handles this CR and generates the following required resources:

- ServiceAccount: new account for `event-tailer`
- ClusterRole: sets the `event-tailer's` roles
- ClusterRoleBinding: links the account with the roles
- ConfigMap: contains the configuration for the `event-tailer` pod
- StatefulSet: manages the lifecycle of the `event-tailer` pod, which uses the `banzaicloud/eventrouter:v0.1.0` image to tail events

## Create event tailer

1. The simplest way to init an `event-tailer` is to create a new `event-tailer` resource with a `name` and `controlNamespace` field specified. The following command creates an event tailer called `sample`:

    ```bash
    kubectl apply -f - <<EOF
    apiVersion: logging-extensions.banzaicloud.io/v1alpha1
    kind: EventTailer
    metadata:
      name: sample
    spec:
      controlNamespace: default
    EOF
    ```

1. Check that the new object has been created by running:

    ```bash
    kubectl get eventtailer
    ```

    Expected output:

    ```bash
    NAME     AGE
    sample   22m
    ```

1. You can see the events in JSON format by checking the log of the `event-tailer` pod. This way Logging operator can collect the events, and handle them as any other log. Run:

    ```bash
    kubectl logs -l app.kubernetes.io/instance=sample-event-tailer | head -1 | jq
    ```

    The output should be similar to:

    ```bash
    {
      "verb": "UPDATED",
      "event": {
        "metadata": {
          "name": "kube-scheduler-kind-control-plane.17145dad77f0e528",
          "namespace": "kube-system",
          "uid": "c2416fa6-7b7f-4a7d-a5f1-b2f2241bd599",
          "resourceVersion": "424",
          "creationTimestamp": "2022-09-13T08:19:22Z",
          "managedFields": [
            {
              "manager": "kube-controller-manager",
              "operation": "Update",
              "apiVersion": "v1",
              "time": "2022-09-13T08:19:22Z"
            }
          ]
        },
        "involvedObject": {
          "kind": "Pod",
          "namespace": "kube-system",
          "name": "kube-scheduler-kind-control-plane",
          "uid": "7bd2c626-84f2-49c3-8e8e-8a7c0514b686",
          "apiVersion": "v1",
          "resourceVersion": "322"
        },
        "reason": "NodeNotReady",
        "message": "Node is not ready",
        "source": {
          "component": "node-controller"
        },
        "firstTimestamp": "2022-09-13T08:19:22Z",
        "lastTimestamp": "2022-09-13T08:19:22Z",
        "count": 1,
        "type": "Warning",
        "eventTime": null,
        "reportingComponent": "",
        "reportingInstance": ""
      },...
    ```

1. Once you have an `event-tailer`, you can bind your events to a specific logging flow. The following example configures a flow to route the previously created `sample-eventtailer` to the `sample-output`.

    ```bash
    kubectl apply -f - <<EOF
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Flow
    metadata:
      name: eventtailer-flow
      namespace: default
    spec:
      filters:
      - tag_normaliser: {}
      match:
      # keeps data matching to label, the rest of the data will be discarded by this flow implicitly
      - select:
          labels:
            app.kubernetes.io/name: sample-event-tailer
      outputRefs:
        - sample-output
    EOF
    ```

## Delete event tailer

To remove an unwanted tailer, delete the related `event-tailer` custom resource. This terminates the `event-tailer` pod. For example, run the following command to delete the event tailer called `sample`:

```bash
kubectl delete eventtailer sample && kubectl get pod
```

Expected output:

```bash
eventtailer.logging-extensions.banzaicloud.io "sample" deleted
NAME                    READY   STATUS        RESTARTS   AGE
sample-event-tailer-0   1/1     Terminating   0          12s
```

## Persist event logs

Event-tailer supports persist mode. In this case, the logs generated from events are stored on a persistent volume. Add the following configuration to your event-tailer spec. In this example, the event tailer is called `sample`:

```bash
kubectl apply -f - <<EOF
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: EventTailer
metadata:
  name: sample
spec:
  controlNamespace: default
  positionVolume:
    pvc:
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
        volumeMode: Filesystem
EOF
```

Logging operator manages the persistent volume of event-tailer automatically, you don't have any further task with it. To check that the persistent volume has been created, run:

```bash
kubectl get pvc && kubectl get pv
```

The output should be similar to:

```bash
NAME                                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
sample-event-tailer-sample-event-tailer-0   Bound    pvc-6af02cb2-3a62-4d24-8201-dc749034651e   1Gi        RWO            standard       43s
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                               STORAGECLASS   REASON   AGE
pvc-6af02cb2-3a62-4d24-8201-dc749034651e   1Gi        RWO            Delete           Bound    default/sample-event-tailer-sample-event-tailer-0   standard                42s
```

## Configuration options

For the detailed list of configuration options, see the [EventTailer CRD reference]({{< relref "../crds/extensions/v1alpha1/eventtailer_types.md" >}}).
