---
title: Troubleshooting Fluentd
shorttitle: Fluentd
weight: 200
---

<p align="center"><img src="/docs/one-eye/logging-operator/img/fluentd.png" height="100"></p>

The following sections help you troubleshoot the Fluentd statefulset component of the Logging operator.

## Check Fluentd pod status (statefulset)

Verify that the Fluentd statefulset is available using the following command: `kubectl get statefulsets`

Expected output:

```bash
NAME                   READY   AGE
logging-demo-fluentd   1/1     1m
```

## ConfigCheck

The Logging operator has a builtin mechanism that validates the generated fluentd configuration before applying it to fluentd. You should be able to see the configcheck pod and it's log output. The result of the check is written into the `status` field of the corresponding `Logging` resource.

In case the operator is stuck in an error state caused by a failed configcheck, restore the previous configuration by modifying or removing the invalid resources to the point where the configcheck pod is finally able to complete successfully.

## Check Fluentd configuration

Use the following command to display the configuration of Fluentd:
`kubectl get secret logging-demo-fluentd-app -o jsonpath="{.data['fluentd\.conf']}" | base64 --decode`

The output should be similar to the following:

```xml
<source>
  @type forward
  @id main_forward
  bind 0.0.0.0
  port 24240
  <transport tls>
    ca_path /fluentd/tls/ca.crt
    cert_path /fluentd/tls/tls.crt
    client_cert_auth true
    private_key_path /fluentd/tls/tls.key
    version TLSv1_2
  </transport>
  <security>
    self_hostname fluentd
    shared_key Kamk2_SukuWenk
  </security>
</source>
<match **>
  @type label_router
  @id main_label_router
  <route>
    @label @427b3e18f3a3bc3f37643c54e9fc960b
    labels app.kubernetes.io/instance:logging-demo,app.kubernetes.io/name:log-generator
    namespace logging
  </route>
</match>
<label @427b3e18f3a3bc3f37643c54e9fc960b>
  <match kubernetes.**>
    @type tag_normaliser
    @id logging-demo-flow_0_tag_normaliser
    format ${namespace_name}.${pod_name}.${container_name}
  </match>
  <filter **>
    @type parser
    @id logging-demo-flow_1_parser
    key_name log
    remove_key_name_field true
    reserve_data true
    <parse>
      @type nginx
    </parse>
  </filter>
  <match **>
    @type s3
    @id logging_logging-demo-flow_logging-demo-output-minio_s3
    aws_key_id WVKblQelkDTSKTn4aaef
    aws_sec_key LAmjIah4MTKTM3XGrDxuD2dTLLmysVHvZrtxpzK6
    force_path_style true
    path logs/${tag}/%Y/%m/%d/
    s3_bucket demo
    s3_endpoint http://logging-demo-minio.logging.svc.cluster.local:9000
    s3_region test_region
    <buffer tag,time>
      @type file
      path /buffers/logging_logging-demo-flow_logging-demo-output-minio_s3.*.buffer
      retry_forever true
      timekey 10s
      timekey_use_utc true
      timekey_wait 0s
    </buffer>
  </match>
</label>
```

## Set Fluentd log Level

Use the following command to change the log level of Fluentd.
`kubectl edit loggings.logging.banzaicloud.io logging-demo`

```yaml
fluentd:
  logLevel: debug
```

## Get Fluentd logs

The following command displays the logs of the Fluentd container.
`kubectl exec -it logging-demo-fluentd-0 cat /fluentd/log/out`

> The [One Eye](/products/one-eye/) observability tool can [display Fluentd logs on its web UI](/docs/one-eye/troubleshooting/), where you can select which replica to inspect, search the logs, and use other ways to monitor and troubleshoot your logging infrastructure.

> Tip: If the logs include the `error="can't create buffer file ...` error message, FluentD can’t create the buffer file at the specified location. This can mean for example that the disk is full, the filesystem is read-only, or some other permission error. Check the buffer-related settings of your [Fluentd configuration](/docs/one-eye/logging-operator/configuration/fluentd/).

## Set stdout as an output

You can use an stdout filter at any point in the flow to dump the log messages to the stdout of the Fluentd container. For example:
`kubectl edit loggings.logging.banzaicloud.io logging-demo`

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: exchange
  namespace: logging
spec:
  filters:
    - stdout: {}
  localOutputRefs:
    - exchange
  selectors:
    application: exchange
```

## Check the buffer path in the fluentd container

`kubectl exec -it logging-demo-fluentd-0 ls  /buffers`

```bash
Defaulting container name to fluentd.
Use 'kubectl describe pod/logging-demo-fluentd-0 -n logging' to see all of the containers in this pod.
logging_logging-demo-flow_logging-demo-output-minio_s3.b598f7eb0b2b34076b6da13a996ff2671.buffer
logging_logging-demo-flow_logging-demo-output-minio_s3.b598f7eb0b2b34076b6da13a996ff2671.buffer.meta
```

{{< include-headless "support-troubleshooting.md" "one-eye/logging-operator" >}}
