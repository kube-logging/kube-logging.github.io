---
title: Namespace-based multitenancy
weight: 600
---

Namespace-based multitenancy allows you have multiple tenants (for example, different developer teams) on the same cluster who can configure their own logging resources within their assigned namespaces.

- These resources are separate from the resources of the other tenants, and
- the configuration checks of Logging operator ensure that errors in the resources of a namespace don't affect other tenants. For example, an error cannot bring down the logging of the entire cluster.


## Sample setup

The following procedure creates two tenants (A and B) and their respective namespaces on a two-node cluster.

1. If you don't already have a cluster, create one with your provider. For a quick test, you can use a local cluster, for example, using minikube:

    ```bash
    minikube start --nodes=2
    ```

1. Set labels on the nodes that correspond to your tenants, for example, `tenant-a` and `tenant-b`.

    ```shell
    kubectl label node minikube tenant=tenant-a
    kubectl label node minikube-m02 tenant=tenant-b
    ```

1. Apply the sample resources from the [project repository](https://github.com/kube-logging/logging-operator/tree/master/config/samples/mulitenant-hard/logging). These create namespaces, flows, and sample outputs for the two tenants.
    <!-- FIXME Describe what these resources do / how they are configured -->

    ```bash
    kubectl apply -f https://github.com/kube-logging/logging-operator/tree/master/config/samples/mulitenant-hard/logging
    ```
    <!-- FIXME check if applying a folder via http works, probably not -->

1. (Optional) Install a sample log generator application to the respective namespaces of your tenants. For example:

    ```bash
    helm upgrade --install --namespace a --create-namespace --set "nodeSelector.tenant=tenant-a" log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator
    helm upgrade --install --namespace b --create-namespace --set "nodeSelector.tenant=tenant-b" log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator
    ```

1. Check that your pods are up and running by running `kubectl get pods`

    If you have followed the examples, the output should look like:

    ```bash
    NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE     IP             NODE           NOMINATED NODE   READINESS GATES
    a-control     a-fluentbit-2997s                  1/1     Running   0             9m15s   10.244.0.5     minikube       <none>           <none>
    a-control     a-fluentd-0                        2/2     Running   0             9m15s   10.244.0.6     minikube       <none>           <none>
    a             log-generator-6cfb45c684-kbzk4     1/1     Running   0             11m     10.244.0.3     minikube       <none>           <none>
    b-control     b-fluentbit-9bvbn                  1/1     Running   0             7m30s   10.244.1.7     minikube-m02   <none>           <none>
    b-control     b-fluentd-0                        2/2     Running   0             7m29s   10.244.1.8     minikube-m02   <none>           <none>
    b             log-generator-7b95b6fdc5-62bnr     1/1     Running   0             11m     10.244.1.3     minikube-m02   <none>           <none>
    ```
