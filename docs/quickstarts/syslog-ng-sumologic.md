---
title: Sumo Logic with Logging operator and syslog-ng
shorttitle: Sumo Logic with syslog-ng
weight: 300
---

This guide helps you install and configure the Logging operator and syslog-ng to forward logs to your Sumo Logic account.

## Prerequisites

We assume that you already have:

- A Sumo Logic account.
- A [HTTP Hosted Collector](https://help.sumologic.com/03Send-Data/Sources/02Sources-for-Hosted-Collectors/HTTP-Source) configured in the Sumo Logic service.

    To configure a Hosted Collector, complete the steps in the [Configure a Hosted Collector](https://help.sumologic.com/03Send-Data/Hosted-Collectors/Configure-a-Hosted-Collector) section on the official Sumo Logic website.

- The unique HTTP collector code you receive while configuring your Host Collector for HTTP requests.

--------------

## Deploy the Logging operator and a demo Application

Install the Logging operator and a demo application to provide sample log messages.

### Deploy the Logging operator with Helm {#helm}

{{< include-headless "deploy-helm-intro.md" "one-eye/logging-operator" >}}

1. Add the chart repository of the Logging operator using the following commands:

    ```bash
    helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com
    helm repo update
    ```

1. Install the demo application and its logging definition.

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-demo banzaicloud-stable/logging-demo \
      --set "loki.enabled=True"
    ```

1. [Validate your deployment](#validate).

## Configure the Logging operator

1. Create logging `Namespace`

    ```bash
    kubectl create ns logging
    ```

1. Create the `logging` resource.

    ```bash
    kubectl -n logging apply -f - <<"EOF"
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

    > Note: You can use the `ClusterOutput` and `ClusterFlow` resources only in the `controlNamespace`.

1. Create a Sumo Logic output secret from the URL of your Sumo Logic collection.

    ```bash
    kubectl create secret generic logging-sumo -n logging --from-literal "sumoURL=https://endpoint1.collection.eu.sumologic.com/......"
    ```

1. Create a `ClusterOutput` resource.

    ```bash
    kubectl -n logging apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: ClusterOutput
    metadata:
      name: sumologic-syslog-ng-output
    spec:
      sumologic:
        buffer:
          flush_interval: 10s
          flush_mode: interval
        endpoint:
          valueFrom:
            secretKeyRef:
              name:  logging-sumo
              key: sumoURL
        source_name: kubernetes
    EOF
    ```

1. Create a `SyslogNGFlow` resource.

    ```bash
    kubectl -n logging apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: SyslogNGFlow
    metadata:
      name: TestFlow
      namespace: default
    spec:
      match:
        and:
        - regexp:
            value: json.kubernetes.labels.app.kubernetes.io/instance
            pattern: one-eye-log-generator
            type: string
        - regexp:
            value:  json.kubernetes.labels.app.kubernetes.io/name
            pattern: log-generator
            type: string
      filters:
      -  parser:
          regexp: 
            patterns:
            - '^(?<remote>[^ ]*) (?<host>[^ ]*) (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)"(?:\s+(?<http_x_forwarded_for>[^ ]+))?)?$'
            template: ${json.message}
            prefix: json.
      - rewrite:
        -  set:
            field: json.cluster
            value: xxxxx
        -  unset:
            field: json.message
        -  set:
            field: json.source
            value: /var/log/log-generator
            condition:
              regexp:
                value:  json.kubernetes.container_name
                pattern: log-generator
                type: string
      localOutputRefs:
        - sumologic-syslog-ng-output
    ```

1. [Validate your deployment](#validate).

## Validate the deployment {#validate}

Check the output. The logs will be available in the bucket on a `path` like:

```bash
/logs/default.default-logging-simple-fluentbit-lsdp5.fluent-bit/2019/09/11/201909111432_0.gz
```

{{< include-headless "note-troubleshooting.md" "one-eye/logging-operator" >}}
