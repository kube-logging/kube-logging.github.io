---
title: Nodegroup-based multitenancy
weight: 600
---

Nodegroup-based multitenancy allows you to have multiple tenants (for example, different developer teams or customer environments) on the same cluster who can configure their own logging resources within their assigned namespaces residing on different node groups.
These resources are isolated from the resources of the other tenants so the configuration issues and performance characteristics of one tenant doesn't affect the others.

## Sample setup

The following procedure creates two tenants (A and B) and their respective namespaces on a two-node cluster.

1. If you don't already have a cluster, create one with your provider. For a quick test, you can use a local cluster, for example, using minikube:

    ```bash
    minikube start --nodes=2
    ```

1. Set labels on the nodes that correspond to your tenants, for example, `tenant-a` and `tenant-b`.

    ```bash
    kubectl label node minikube tenant=tenant-a
    kubectl label node minikube-m02 tenant=tenant-b
    ```

1. Install the logging operator

    ```bash
    helm install logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
    ```

1. Apply the sample resources from the [project repository](https://github.com/kube-logging/logging-operator/tree/master/config/samples/mulitenant-hard/logging). These create namespaces, flows, and sample outputs for the two tenants.

1. (Optional) Install a sample log generator application to the respective namespaces of your tenants. For example:

    ```bash
    helm upgrade --install --namespace a --create-namespace --set "nodeSelector.tenant=tenant-a" log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator
    helm upgrade --install --namespace b --create-namespace --set "nodeSelector.tenant=tenant-b" log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator
    ```

1. Check that your pods are up and running by running `kubectl get pods -A`

    If you have followed the examples, the output should look like:

    ```bash
    NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
    a-control     a-fluentbit-4tqzg                  1/1     Running   0             9m29s
    a-control     a-fluentd-0                        2/2     Running   0             4m48s
    a             log-generator-6cfb45c684-q6fl6     1/1     Running   0             3m25s
    b-control     b-fluentbit-qmf58                  1/1     Running   0             9m20s
    b-control     b-fluentd-0                        2/2     Running   0             9m16s
    b             log-generator-7b95b6fdc5-cshh7     1/1     Running   0             8m49s
    default       logging-operator-bbd66bb7d-qvsmg   1/1     Running   0             35m
    infra         test-receiver-7c45f9cd77-whvlv     1/1     Running   0             53m
    ```

1. Check logs coming from both tenants `kubectl logs -f -n infra svc/test-receiver`

    Expected output should show logs from both tenants
    ```bash
    [0] tenant_a: [[1695999280.157810965, {}], {"log"=>"15.238.250.48 - - [29/Sep/2023:14:54:38 +0000] "PUT /pro...
    [0] tenant_b: [[1695999280.160868923, {}], {"log"=>"252.201.89.36 - - [29/Sep/2023:14:54:33 +0000] "POST /bl...
    ```
