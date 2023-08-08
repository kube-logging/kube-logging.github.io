---
title: Sumo Logic with Logging operator and syslog-ng
linktitle: Sumo Logic with syslog-ng
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

{{< include-headless "deploy-helm-intro.md" >}}

1. Install the logging-operator

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
    ```

## Configure the Logging operator

1. Create the `logging` resource with a persistent syslog-ng installation.

    ```bash
    kubectl apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Logging
    metadata:
      name: demo
    spec:
      controlNamespace: logging
      fluentbit: {}
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
    EOF
    ```

    > Note: You can use the `ClusterOutput` and `ClusterFlow` resources only in the `controlNamespace`.

1. Create a Sumo Logic output secret from the URL of your Sumo Logic collection.

    ```bash
    kubectl create secret generic sumo-collector -n logging --from-literal "token=XYZ"
    ```

1. Create a `SyslogNGOutput` resource.

    ```bash
    kubectl -n logging apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: SyslogNGOutput
    metadata:
      name: sumologic-syslog-ng-output
    spec:
      sumologic-http: 
        collector:
          valueFrom:
            secretKeyRef:
              key: token
              name: sumo-collector
        deployment: us2
        batch-lines: 1000
        disk_buffer:
          disk_buf_size: 512000000
          dir: /buffers
          reliable: true
        body: "$(format-json --subkeys json. --exclude json.kubernetes.annotations.* json.kubernetes.annotations=literal($(format-flat-json --subkeys json.kubernetes.annotations.)) --exclude json.kubernetes.labels.* json.kubernetes.labels=literal($(format-flat-json --subkeys json.kubernetes.labels.)))"
        headers:
          - 'X-Sumo-Name: source-name'
          - 'X-Sumo-Category: source-category'
        tls:
          use-system-cert-store: true
    EOF
    ```

1. Create a `SyslogNGFlow` resource.

    ```bash
    kubectl -n logging apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: SyslogNGFlow
    metadata:
      name: log-generator
    spec:
      match:
        and:
        - regexp:
            value: json.kubernetes.labels.app.kubernetes.io/instance
            pattern: log-generator
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
    EOF
    ```

1. Install log-generator to produce logs with the label `app.kubernetes.io/name: log-generator`

     ```bash
     helm upgrade --install --wait --create-namespace --namespace logging log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator
     ```

{{< include-headless "note-troubleshooting.md" >}}
