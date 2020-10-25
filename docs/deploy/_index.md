---
title: Install the Logging operator
shorttitle: Deployment
weight: 200
---



> Caution: The **master branch** is under heavy development. Use [releases](https://github.com/banzaicloud/logging-operator/releases) instead of the master branch to get stable software.

## Prerequisites

- Logging operator requires Kubernetes v1.14.x or later.
- For the [Helm-based installation](#deploy-with-helm) you need Helm v3.21.0 or later.

## Deploy the Logging operator with One Eye {#deploy-with-one-eye}

If you are using the [One Eye observability tool](/products/one-eye/), use the [one-eye command line tool](/docs/one-eye/cli/) to [install the Logging operator](/docs/one-eye/cli/reference/one-eye_logging_install/).

```bash
one-eye logging install
```

After that, you can configure your logging flows and outputs using the:

- [One Eye web interface](/docs/one-eye/configure-logging-infrastructure/configuration-overview/),
- [one-eye command line tool](/docs/one-eye/cli/reference/one-eye_logging_configure/), or
- declaratively using the [Observer custom resource](/docs/one-eye/crds/oneeye_types/).

## Deploy the Logging operator from Kubernetes Manifests {#deploy-with-manifest}

Complete the following steps to deploy the Logging operator using Kubernetes manifests. Alternatively, you can also [install the operator using Helm](#deploy-with-helm).

1. Create a controlNamespace called "logging".

    ```bash
    kubectl create ns logging
    ```

1. Create a ServiceAccount and install cluster roles.

    ```bash
    kubectl -n logging create -f https://raw.githubusercontent.com/banzaicloud/logging-operator-docs/master/docs/deploy/manifests/rbac.yaml
    ```

1. Apply the ClusterResources.

    ```bash
    kubectl -n logging create -f https://raw.githubusercontent.com/banzaicloud/logging-operator/master/config/crd/bases/logging.banzaicloud.io_clusterflows.yaml
    kubectl -n logging create -f https://raw.githubusercontent.com/banzaicloud/logging-operator/master/config/crd/bases/logging.banzaicloud.io_clusteroutputs.yaml
    kubectl -n logging create -f https://raw.githubusercontent.com/banzaicloud/logging-operator/master/config/crd/bases/logging.banzaicloud.io_flows.yaml
    kubectl -n logging create -f https://raw.githubusercontent.com/banzaicloud/logging-operator/master/config/crd/bases/logging.banzaicloud.io_loggings.yaml
    kubectl -n logging create -f https://raw.githubusercontent.com/banzaicloud/logging-operator/master/config/crd/bases/logging.banzaicloud.io_outputs.yaml
    ```

1. Deploy the Logging operator.

    ```bash
    kubectl -n logging create -f https://raw.githubusercontent.com/banzaicloud/logging-operator-docs/master/docs/deploy/manifests/deployment.yaml
    ```

## Deploy Logging operator with Helm {#deploy-with-helm}

<p align="center"><img src="../img/helm.svg" width="150"></p>
<p align="center">

Complete the following steps to deploy the Logging operator using Helm. Alternatively, you can also [install the operator using Kubernetes manifests](#deploy-with-manifest).

> Note: For the Helm-based installation you need Helm v3.21.0 or later.

1. Add operator chart repository.

    ```bash
    helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com
    helm repo update
    ```

2. Install the Logging Operator

    - Helm v3

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging --name logging-operator banzaicloud-stable/logging-operator \
      --set createCustomResource=false"
    ```

        > You can install the `logging` resource with built-in TLS generation using a [Helm chart](https://github.com/banzaicloud/logging-operator/tree/master/charts/logging-operator-logging).

## Check the Logging operator deployment

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
