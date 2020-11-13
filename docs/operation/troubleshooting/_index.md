---
title: Logging operator troubleshooting
shorttitle: Troubleshooting
weight: 400
aliases:
    - /docs/one-eye/logging-operator/troubleshooting/
---

<p align="center"><img src="/docs/one-eye/logging-operator/img/troubleshooting.svg" width="260"></p>
<p align="center">

The following tips and commands can help you to troubleshoot your Logging operator installation.

## First things to do

1. Check that the necessary CRDs are installed. Issue the following command: `kubectl get crd`
   The output should include the following CRDs:

    ```bash
    clusterflows.logging.banzaicloud.io     2019-12-05T15:11:48Z
    clusteroutputs.logging.banzaicloud.io   2019-12-05T15:11:48Z
    flows.logging.banzaicloud.io            2019-12-05T15:11:48Z
    loggings.logging.banzaicloud.io         2019-12-05T15:11:48Z
    outputs.logging.banzaicloud.io          2019-12-05T15:11:48Z
    ```

1. Verify that the Logging operator pod is running. Issue the following command: `kubectl get pods |grep logging-operator`
   The output should include the a running pod, for example:

    ```bash
    NAME                                          READY   STATUS      RESTARTS   AGE
    logging-demo-log-generator-6448d45cd9-z7zk8   1/1     Running     0          24m
    ```

After that, check the following sections for further tips.

{{< toc >}}

{{< include-headless "support-troubleshooting.md" "one-eye/logging-operator" >}}
