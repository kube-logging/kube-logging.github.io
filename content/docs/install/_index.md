---
title: Install the Logging operator
shorttitle: Install
weight: 100
aliases:
    - /docs/one-eye/logging-operator/deploy/
---

> Caution: The **master branch** is under heavy development. Use [releases](https://github.com/banzaicloud/logging-operator/releases) instead of the master branch to get stable software.

## Prerequisites

- Logging operator requires Kubernetes v1.14.x or later.
- For the [Helm-based installation](#helm) you need Helm v3.2.1 or later.

## Deploy the Logging operator with One Eye {#deploy-with-one-eye}

If you are using the [One Eye observability tool](/products/one-eye/), use the [one-eye command line tool](/docs/one-eye/cli/) to [install the Logging operator](/docs/one-eye/cli/reference/one-eye_logging_install/).

```bash
one-eye logging install
```

After that, you can configure your logging flows and outputs using the:

- [One Eye web interface](/docs/one-eye/configure-logging-infrastructure/configuration-overview/),
- [one-eye command line tool](/docs/one-eye/cli/reference/one-eye_logging_configure/), or
- declaratively using the [Observer custom resource](/docs/one-eye/crds/oneeye_types/).

## Deploy Logging operator with Helm {#helm}

<p align="center"><img src="../img/helm.svg" width="150"></p>
<p align="center">

{{< include-headless "deploy-helm-intro.md" "one-eye/logging-operator" >}}

1. Add the chart repository of the Logging operator using the following commands:

    ```bash
    helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com
    helm repo update
    ```

1. Install the Logging operator.

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-operator banzaicloud-stable/logging-operator
    ```

    > You can install the `logging` resource with built-in TLS generation using the [Helm chart](https://github.com/banzaicloud/logging-operator/tree/master/charts/logging-operator-logging).


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
