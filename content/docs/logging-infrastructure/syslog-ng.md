---
title: Configure syslog-ng
linktitle: syslog-ng log forwarder
weight: 300
---

{{< include-headless "syslog-ng-minimum-version.md" >}}

This page shows some examples on configuring syslog-ng.

## Ways to configure syslog-ng

There are two ways to configure the syslog-ng statefulset:

1. Using the **spec.syslogNG** section of {{% xref "/docs/logging-infrastructure/logging.md" %}}.
1. Using the standalone syslogNGConfig CRD. This method is only available in Logging operator version 4.5 and newer, and the specification of the CRD is compatible with the **spec.syslogNG** configuration method. That way you can use a multi-tenant model, where tenant owners are responsible for operating their own aggregator, while the Logging resource is in control of the central operations team.

    The standalone syslogNGConfig is a namespaced resource that allows the configuration of the syslog-ng aggregator in the control namespace, separately from the Logging resource. This allows you to use a multi-tenant model, where tenant owners are responsible for operating their own aggregator, while the Logging resource is in control of the central operations team. For more information about the multi-tenancy model where the collector is capable of routing logs based on namespaces to individual aggregators and where aggregators are fully isolated, see this blog post about [Multi-tenancy using Logging operator](https://axoflow.com/multi-tenancy-using-logging-operator/).

For the detailed list of available parameters, see {{% xref "/docs/configuration/crds/v1beta1/syslogng_types.md" %}}.

### Migrating from **spec.syslogNG** to syslogNGConfig {#migrating}

The standalone syslogNGConfig CRD is only available in Logging operator version 4.5 and newer. Its specification and logic is identical with the **spec.syslogNG** configuration method. Using the syslogNGConfig CRD allows you to remove the **spec.syslogNG** section from the Logging CRD, which has the following benefits.

- RBAC control over the syslogNGConfig CRD, so you can have separate roles that can manage the Logging resource and the syslogNGConfig resource (that is, the syslog-ng deployment).
- It reduces the size of the Logging resource, which can grow big enough to reach the annotation size limit in certain scenarios (e.g. when using `kubectl apply`).
- You can use a multi-tenant model, where tenant owners are responsible for operating their own aggregator, while the Logging resource is in control of the central operations team.

To migrate your **spec.syslogNG** configuration from the Logging resource to a separate syslogNGConfig CRD, complete the following steps.

1. Open your Logging resource and find the **spec.syslogNG** section. For example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: example-logging-resource
    spec:
      controlNamespace: logging
      syslogNG:
        ...
    ```

1. Create a new syslogNGConfig CRD. For the value of **metadata.name**, use the name of the Logging resource, for example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: syslogNGConfig
    metadata:
      # Use the name of the logging resource
      name: example-logging-resource
      # Use the control namespace of the logging resource
      namespace: logging
    ```

1. Copy the the **spec.syslogNG** section from the Logging resource into the **spec** section of the syslogNGConfig CRD, then fix the indentation. For example:

    ```yaml
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: syslogNGConfig
    metadata:
      # Use the name of the logging resource
      name: example-logging-resource
      # Use the control namespace of the logging resource
      namespace: logging
    spec:
      ...
    ```

1. Delete the **spec.syslogNG** section from the Logging resource, then apply the Logging and the syslogNGConfig CRDs.

## Using the standalone syslogNGConfig resource

The standalone syslogNGConfig is a namespaced resource that allows the configuration of the syslog-ng aggregator in the control namespace, separately from the Logging resource. This allows you to use a multi-tenant model, where tenant owners are responsible for operating their own aggregator, while the Logging resource is in control of the central operations team. For more information about the multi-tenancy model where the collector is capable of routing logs based on namespaces to individual aggregators and where aggregators are fully isolated, see this blog post about [Multi-tenancy using Logging operator](https://axoflow.com/multi-tenancy-using-logging-operator/).

A `Logging` resource can have only one `syslogNGConfig` at a time. The controller registers the active `syslogNGConfig` resource into the `Logging` resource's status under `syslogNGConfigName`, and also registers the `Logging` resource name under `logging` in the `syslogNGConfig` resource's status, for example:

```shell
kubectl get logging example -o jsonpath='{.status}' | jq .
{
  "configCheckResults": {
    "ac2d4553": true
  },
  "syslogNGConfigName": "example"
}
```

```shell
kubectl get syslogngconfig example -o jsonpath='{.status}' | jq .
{
  "active": true,
  "logging": "example"
}
```

If there is a conflict, the controller adds a problem to both resources so that both the operations team and the tenant users can notice the problem. For example, if a `syslogNGConfig` is already registered to a `Logging` resource and you create another `syslogNGConfig` resource in the same namespace, then the first `syslogNGConfig` is left intact, while the second one should have the following status:

```shell
kubectl get syslogngconfig example2 -o jsonpath='{.status}' | jq .
{
  "active": false,
  "problems": [
    "logging already has a detached syslog-ng configuration, remove excess configuration objects"
  ],
  "problemsCount": 1
}
```

The `Logging` resource will also show the issue:

```shell
kubectl get logging example -o jsonpath='{.status}' | jq .
{
  "configCheckResults": {
    "ac2d4553": true
  },
  "syslogNGConfigName": "example",
  "problems": [
    "multiple syslog-ng configurations found, couldn't associate it with logging"
  ],
  "problemsCount": 1
}
```

## Volume mount for buffering

The following example sets a volume mount that syslog-ng can use for buffering messages on the disk (if {{% xref "/docs/configuration/plugins/syslog-ng-outputs/disk_buffer.md" %}} is configured in the output).

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
  name: test
spec:
  syslogNG:
    statefulSet:
      spec:
        template:
          spec:
            containers:
            - name: syslog-ng
              volumeMounts:
              - mountPath: /buffers
                name: buffer
        volumeClaimTemplates:
        - metadata:
            name: buffer
          spec:
            accessModes:
            - ReadWriteOnce
            resources:
              requests:
                storage: 10Gi
```

{{< include-headless "cpu-memory-requirements.md" >}}

## Probe

A [Probe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) is a diagnostic performed periodically by the kubelet on a Container. To perform a diagnostic, the kubelet calls a Handler implemented by the Container. You can configure a probe for syslog-ng in the **livenessProbe** section of the {{% xref "/docs/logging-infrastructure/logging.md" %}}. For example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging-simple
spec:
  syslogNG:
    livenessProbe:
      periodSeconds: 60
      initialDelaySeconds: 600
      exec:
        command:
        - "/usr/sbin/syslog-ng-ctl"
        - "--control=/tmp/syslog-ng/syslog-ng.ctl"
        - "query"
        - "get"
        - "global.sdata_updates.processed"
  controlNamespace: logging
```

You can use the following parameters:

| Name                    | Type           | Default | Description |
|-------------------------|----------------|---------|-------------|
| initialDelaySeconds | int | 30 | Number of seconds after the container has started before liveness probes are initiated. |
| timeoutSeconds | int | 0 | Number of seconds after which the probe times out. |
| periodSeconds | int | 10 | How often (in seconds) to perform the probe. |
| successThreshold | int | 0 | Minimum consecutive successes for the probe to be considered successful after having failed. |
| failureThreshold | int | 3 |  Minimum consecutive failures for the probe to be considered failed after having succeeded. |
| exec | array | {} |  Exec specifies the action to take. [More info](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#execaction-v1-core) |

> Note: To configure readiness probes, see {{% xref "/docs/operation/readiness-probe.md" %}}.
