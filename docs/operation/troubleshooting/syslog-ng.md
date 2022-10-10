---
title: Troubleshooting syslog-ng
shorttitle: syslog-ng
weight: 200
---

The following sections help you troubleshoot the syslog-ng statefulset component of the Logging operator.

## Check syslog-ng pod status (statefulset)

Verify that the syslog-ng statefulset is available using the following command: `kubectl get statefulsets`

Expected output:

```bash
NAME                   READY   AGE
logging-demo-syslogng   1/1     1m
```

## ConfigCheck

The Logging operator has a builtin mechanism that validates the generated syslog-ng configuration before applying it to syslog-ng. You should be able to see the configcheck pod and its log output. The result of the check is written into the `status` field of the corresponding `Logging` resource.

In case the operator is stuck in an error state caused by a failed configcheck, restore the previous configuration by modifying or removing the invalid resources to the point where the configcheck pod is finally able to complete successfully.

## Check syslog-ng configuration

Use the following command to display the configuration of syslog-ng:
`kubectl get secret logging-demo-syslogng-app -o jsonpath="{.data['syslogng\.conf']}" | base64 --decode`
<!-- 
FIXME
The output should be similar to the following:

```xml

``` -->

<!-- ## Set syslog-ng log Level

Use the following command to change the log level of syslog-ng.
`kubectl edit loggings.logging.banzaicloud.io logging-demo`

```yaml
syslogNG:
  logLevel: debug
```

## Get syslog-ng logs

The following command displays the logs of the syslog-ng container.
`kubectl exec -it logging-demo-fluentd-0 cat /syslog-ng/log/out` -->

{{< include-headless "support-troubleshooting.md" "one-eye/logging-operator" >}}
