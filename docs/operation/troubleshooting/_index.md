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

    <!-- FIXME syslog-ng-sek is letrejonnek automatikusan? -->

1. Verify that the Logging operator pod is running. Issue the following command: `kubectl get pods |grep logging-operator`
   The output should include the a running pod, for example:

    ```bash
    NAME                                          READY   STATUS      RESTARTS   AGE
    logging-demo-log-generator-6448d45cd9-z7zk8   1/1     Running     0          24m
    ```

1. Check the status of your resources. Beginning with Logging Operator 3.8, all custom resources have a `Status` and a `Problems` field. In a healthy system, the Problems field of the resources is empty, for example:

    ```bash
    kubectl get clusteroutput -A
    ```

    Sample output:

    ```bash
    NAMESPACE   NAME      ACTIVE   PROBLEMS
    default     nullout   true
    ```

    The `ACTIVE` column indicates that the `ClusterOutput` has successfully passed the `configcheck` and presented it in the current fluentd configuration. When no errors are reported the `PROBLEMS` column is empty.

    Take a look at another example, in which we have an incorrect `ClusterFlow`.

    ```bash
    kubectl get clusterflow -o wide
    ```

    Sample output:

    ```bash
    NAME      ACTIVE   PROBLEMS
    all-log   true
    nullout   false    1
    ```

    You can see that the **nullout** `Clusterflow` is inactive and there is *1* problem with the configuration. To display the problem, check the `status` field of the object, for example:

    ```bash
    kubectl get clusterflow nullout -o=jsonpath='{.status}' | jq
    ```

    Sample output:

    ```bash
    {
    "active": false,
    "problems": [
        "dangling global output reference: nullout2"
    ],
    "problemsCount": 1
    }
    ```

After that, check the following sections for further tips.

{{< toc >}}

{{< include-headless "support-troubleshooting.md" "one-eye/logging-operator" >}}
