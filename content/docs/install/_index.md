---
title: Install
weight: 100
aliases:
    - /docs/one-eye/logging-operator/deploy/
---

> Caution: The **master branch** is under heavy development. Use [releases](https://github.com/kube-logging/logging-operator/releases) instead of the master branch to get stable software.

## Prerequisites

- Logging operator requires Kubernetes v1.22.x or later.
- For the [Helm-based installation](#helm) you need Helm v3.8.1 or later.

> With the 4.3.0 release, the chart is now distributed through an OCI registry. <br>
  For instructions on how to interact with OCI registries, please take a look at [Use OCI-based registries](https://helm.sh/docs/topics/registries/).
  For instructions on installing the previous 4.2.3 version, see [Installation for 4.2](/4.2/docs/install). 


## Deploy Logging operator with Helm {#helm}

<p align="center"><img src="../img/helm.svg" alt="Logos" width="150"></p>
<p align="center">

{{< include-headless "deploy-helm-intro.md" >}}

1. {{< include-headless "helm-install-logging-operator.md" >}}

    {{< include-headless "note-helm-chart-logging-resource.md" >}}

## Validate the deployment {#validate}

To verify that the installation was successful, complete the following steps.

1. Check the status of the pods. You should see a new logging-operator pod.

    ```bash
    kubectl -n logging get pods
    ```

    Expected output:

    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    logging-operator-5df66b87c9-wgsdf   1/1     Running   0          21s
    ```

1. Check the CRDs. You should see the following five new CRDs.

    ```bash
    kubectl get crd
    ```

    Expected output:

    ```
    NAME                                    CREATED AT
    clusterflows.logging.banzaicloud.io              2023-08-10T12:05:04Z
    clusteroutputs.logging.banzaicloud.io            2023-08-10T12:05:04Z
    eventtailers.logging-extensions.banzaicloud.io   2023-08-10T12:05:04Z
    flows.logging.banzaicloud.io                     2023-08-10T12:05:04Z
    fluentbitagents.logging.banzaicloud.io           2023-08-10T12:05:04Z
    hosttailers.logging-extensions.banzaicloud.io    2023-08-10T12:05:04Z
    loggings.logging.banzaicloud.io                  2023-08-10T12:05:05Z
    nodeagents.logging.banzaicloud.io                2023-08-10T12:05:05Z
    outputs.logging.banzaicloud.io                   2023-08-10T12:05:05Z
    syslogngclusterflows.logging.banzaicloud.io      2023-08-10T12:05:05Z
    syslogngclusteroutputs.logging.banzaicloud.io    2023-08-10T12:05:05Z
    syslogngflows.logging.banzaicloud.io             2023-08-10T12:05:05Z
    syslogngoutputs.logging.banzaicloud.io           2023-08-10T12:05:06Z
    ```
