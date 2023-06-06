---
title: Install
weight: 100
aliases:
    - /docs/one-eye/logging-operator/deploy/
---

> Caution: The **master branch** is under heavy development. Use [releases](https://github.com/kube-logging/logging-operator/releases) instead of the master branch to get stable software.

## Prerequisites

- Logging operator requires Kubernetes v1.14.x or later.
- For the [Helm-based installation](#helm) you need Helm v3.2.1 or later.

## Deploy Logging operator with Helm {#helm}

<p align="center"><img src="../img/helm.svg" width="150"></p>
<p align="center">

{{< include-headless "deploy-helm-intro.md" >}}

1. Add the chart repository of the Logging operator using the following commands:

    ```bash
    helm repo add kube-logging https://kube-logging.dev/helm-charts
    helm repo update
    ```

1. Install the Logging operator.

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-operator kube-logging/logging-operator
    ```

    > You can install the `logging` resource with built-in TLS generation using the [Helm chart](https://github.com/kube-logging/logging-operator/tree/master/charts/logging-operator-logging).

## Validate the deployment {#validate}

To verify that the installation was successful, complete the following steps.

1. Check the status of the pods. You should see a new logging-operator pod.

    ```bash
    $ kubectl -n logging get pods
    NAME                                        READY   STATUS    RESTARTS   AGE
    logging-logging-operator-599c9cf846-5nw2n   1/1     Running   0          52s
    ```

1. Check the CRDs. You should see the following five new CRDs.

    ```bash
    $  kubectl get crd
    NAME                                    CREATED AT
    clusterflows.logging.banzaicloud.io     2019-11-01T21:30:18Z
    clusteroutputs.logging.banzaicloud.io   2019-11-01T21:30:18Z
    flows.logging.banzaicloud.io            2019-11-01T21:30:18Z
    loggings.logging.banzaicloud.io         2019-11-01T21:30:18Z
    outputs.logging.banzaicloud.io          2019-11-01T21:30:18Z
    ```
