---
title: Troubleshooting Fluent Bit
shorttitle: Fluent Bit
weight: 200
---

<p align="center"><img src="/docs/one-eye/logging-operator/img/fluentbit.png" height="100"></p>

The following sections help you troubleshoot the Fluent Bit component of the Logging operator.

## Check the Fluent Bit daemonset

Verify that the Fluent Bit daemonset is available. Issue the following command: `kubectl get daemonsets`
The output should include a Fluent Bit daemonset, for example:

```bash
NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
logging-demo-fluentbit   1         1         1       1            1           <none>          110s
```

## Check the Fluent Bit configuration

You can display the current configuration of the Fluent Bit daemonset using the following command:
`kubectl get secret logging-demo-fluentbit -o jsonpath="{.data['fluent-bit\.conf']}" | base64 --decode`

The output looks like the following:

```yaml
[SERVICE]
    Flush        1
    Daemon       Off
    Log_Level    info
    Parsers_File parsers.conf
    storage.path  /buffers

[INPUT]
    Name         tail
    DB  /tail-db/tail-containers-state.db
    Mem_Buf_Limit  5MB
    Parser  docker
    Path  /var/log/containers/*.log
    Refresh_Interval  5
    Skip_Long_Lines  On
    Tag  kubernetes.*

[FILTER]
    Name        kubernetes
    Kube_CA_File  /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    Kube_Tag_Prefix  kubernetes.var.log.containers
    Kube_Token_File  /var/run/secrets/kubernetes.io/serviceaccount/token
    Kube_URL  https://kubernetes.default.svc:443
    Match  kubernetes.*
    Merge_Log  On

[OUTPUT]
    Name          forward
    Match         *
    Host          logging-demo-fluentd.logging.svc
    Port          24240

    tls           On
    tls.verify    Off
    tls.ca_file   /fluent-bit/tls/ca.crt
    tls.crt_file  /fluent-bit/tls/tls.crt
    tls.key_file  /fluent-bit/tls/tls.key
    Shared_Key    Kamk2_SukuWenk
    Retry_Limit   False
```

## Debug version of the fluentbit container

All Fluent Bit image tags have a debug version marked with the `-debug` suffix. You can install this debug version using the following command:
`kubectl edit loggings.logging.banzaicloud.io logging-demo`

```yaml
fluentbit:
    image:
      pullPolicy: Always
      repository: fluent/fluent-bit
      tag: 1.3.2-debug
```

After deploying the debug version, you can kubectl exec into the pod using `sh` and look around. For example: `kubectl exec -it logging-demo-fluentbit-778zg sh`

## Check the queued log messages

You can check the buffer directory if Fluent Bit is configured to buffer queued log messages to disk instead of in memory. (You can configure it through the InputTail fluentbit config, by setting the `storage.type` field to `filesystem`.)

`kubectl exec -it logging-demo-fluentbit-9dpzg ls /buffers`

{{< include-headless "support-troubleshooting.md" "one-eye/logging-operator" >}}
