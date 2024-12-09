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

## Image and chart verification {#verify}

Images and charts are signed with GitHub Actions OIDC token. We sign the digests of the images and the charts to ensure the integrity and authenticity of the artifacts.

To verify signatures, you must have [cosign](https://docs.sigstore.dev/cosign/system_config/installation/) installed.

### Image verification

You can verify our images by running the following command.

```shell
cosign verify "ghcr.io/kube-logging/logging-operator@{sha256-IMAGE-DIGEST}" \ 
--certificate-identity "https://github.com/ghcr.io/kube-logging/logging-operator/.github/workflows/artifacts.yaml@{refs/heads/main || refs/tags/<tag_name>}" \ 
--certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

- Replace `{sha256-IMAGE-DIGEST}` with the digest of the image you want to verify. You can find the digests at [ghcr.io/kube-logging/logging-operator](https://ghcr.io/kube-logging/logging-operator). For example, for the 4.11.0 release it's `sha256:50550883905ffe484f210ae65a8e0dbcbc0836c240b1fec454945d8b97830ede`.
- Replace `{refs/heads/main || refs/tags/<tag_name>}` with the reference to the image you want to verify. For example, for the 4.11.0 release, use `refs/tags/4.11.0`.

For example:

```shell
cosign verify "ghcr.io/kube-logging/logging-operator@sha256:50550883905ffe484f210ae65a8e0dbcbc0836c240b1fec454945d8b97830ede" \
    --certificate-identity "https://github.com/kube-logging/logging-operator/.github/workflows/artifacts.yaml@refs/tags/4.11.0" \
    --certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

### Chart verification

You can verify our charts by running the following command.

```shell
cosign verify "ghcr.io/kube-logging/logging-operator@{sha256-CHART-DIGEST}" \ 
--certificate-identity "https://github.com/ghcr.io/kube-logging/logging-operator/.github/workflows/artifacts.yaml@{refs/heads/main || refs/tags/<tag_name>}" \ 
--certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

- Replace `{sha256-CHART-DIGEST}` with the digest of the image you want to verify. You can find the digests at [ghcr.io/kube-logging/helm-charts/logging-operator](https://ghcr.io/kube-logging/helm-charts/logging-operator). For example, for the 4.11.0 release it's `sha256:233407195e1c97382e7fc0dfb00f9c6cadbba2928a64bfce6be072fc37df20eb`.
- Replace `{refs/heads/main || refs/tags/<tag_name>}` with the reference to the image you want to verify. For example, for the 4.11.0 release, use `refs/tags/4.11.0`.

For example:

```shell
cosign verify "ghcr.io/kube-logging/helm-charts/logging-operator@sha256:233407195e1c97382e7fc0dfb00f9c6cadbba2928a64bfce6be072fc37df20eb" \
    --certificate-identity "https://github.com/kube-logging/logging-operator/.github/workflows/artifacts.yaml@refs/tags/4.11.0" \
    --certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```
