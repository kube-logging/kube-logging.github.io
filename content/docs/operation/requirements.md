---
title: CPU and memory requirements
linktitle: Resource requirements
weight: 1300
aliases:
    - /docs/one-eye/logging-operator/requirements/
---

The resource requirements and limits of your Logging operator deployment must match the size of your cluster and the logging workloads. By default, the Logging operator uses the following configuration.

- For **Fluent Bit**:

    ```yaml
    - Limits:
      - cpu: 200m
      - memory: 100M
    - Requests:
      - cpu: 100m
      - memory: 50M
    ```

- For **Fluentd** and **syslog-ng**:

    ```yaml
    - Limits:
      - cpu: 1000m
      - memory: 400M
    - Requests:
      - cpu: 500m
      - memory:  100M
    ```

You can adjust these values in the Logging custom resource, for example:

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Logging
metadata:
  name: default-logging
  namespace: logging
spec:
  fluentd:
    resources:
      requests:
        cpu: 1
        memory: 1Gi
      limits:
        cpu: 2
        memory: 2Gi
  fluentbit:
    resources:
      requests:
        cpu: 500m
        memory: 500M
      limits:
        cpu: 1
        memory: 1Gi
  syslogNG:
    resources:
      requests:
        cpu: 500m
        memory: 500M
      limits:
        cpu: 1
        memory: 1Gi
```
