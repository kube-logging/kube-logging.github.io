---
title: Transport Nginx Access Logs into Kafka with Logging operator
shorttitle: Kafka
weight: 400
---



<p align="center"><img src="../../img/kafka_logo.png" width="340"></p>

This guide describes how to collect application and container logs in Kubernetes using the Logging operator, and how to send them to Kafka.

{{< include-headless "quickstart-figure-intro.md" "one-eye/logging-operator" >}}

<p align="center"><img src="../../img/nignx-kafka.png" width="900"></p>

## Deploy Kafka

> In this demo we are using our kafka operator.
> [Install it with Helm](https://github.com/banzaicloud/kafka-operator#easy-way-installing-with-helm)

## Deploy the Logging operator and a demo Application

Install the Logging operator and a demo application to provide sample log messages.

### Deploy the Logging operator with Helm

To install the Logging operator using Helm, complete these steps. If you want to install the Logging operator using Kubernetes manifests, see [Deploy the Logging operator with Kubernetes manifests]({{< relref "docs/one-eye/logging-operator/install/_index.md#deploy-with-manifest" >}}).

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
   
1. Install the demo application and its logging definition.

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-demo banzaicloud-stable/logging-demo \
      --set "kafka.enabled=True"
    ```

### Deploy the Logging operator with Kubernetes manifests

1. Install the Logging operator. For details, see [How to install Logging-operator from manifests]({{< relref "docs/one-eye/logging-operator/install/_index.md#deploy-with-manifest" >}})
1. Create the `logging` resource.

     ```yaml
     kubectl -n logging apply -f - <<"EOF" 
     apiVersion: logging.banzaicloud.io/v1beta1
     kind: Logging
     metadata:
       name: default-logging-simple
     spec:
       fluentd: {}
       fluentbit: {}
       controlNamespace: logging
     EOF
     ```

     > Note: You can use the `ClusterOutput` and `ClusterFlow` resources only in the `controlNamespace`.

1. Create a Kafka `output` definition.

     ```yaml
    kubectl -n logging apply -f - <<"EOF" 
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Output
    metadata:
      name: kafka-output
    spec:
      kafka:
        brokers: kafka-headless.kafka.svc.cluster.local:29092
        default_topic: topic
        format: 
          type: json    
        buffer:
          tags: topic
          timekey: 1m
          timekey_wait: 30s
          timekey_use_utc: true
    EOF
     ```

     > Note: In production environment, use a longer `timekey` interval to avoid generating too many objects.

1. Create a `flow` resource.

     ```yaml
     kubectl -n logging apply -f - <<"EOF" 
     apiVersion: logging.banzaicloud.io/v1beta1
     kind: Flow
     metadata:
       name: kafka-flow
     spec:
       filters:
         - tag_normaliser: {}
         - parser:
             remove_key_name_field: true
             reserve_data: true
             parse:
               type: nginx
       match:
         - select:
             labels:
               app.kubernetes.io/name: log-generator
       localOutputRefs:
         - kafka-output
     EOF
     ```

1. Install the demo application.

     ```yaml
    kubectl -n logging apply -f - <<"EOF" 
    apiVersion: apps/v1 
    kind: Deployment
    metadata:
      name: log-generator
    spec:
      selector:
        matchLabels:
          app.kubernetes.io/name: log-generator
      replicas: 1
      template:
        metadata:
          labels:   
            app.kubernetes.io/name: log-generator
        spec:
          containers:
          - name: nginx
            image: banzaicloud/log-generator:0.3.2
    EOF
     ```

## Test Your Deployment with kafkacat

1. Exec Kafka test pod

    ```bash
    kubectl -n kafka exec -it kafka-test-c sh
    ```

1. Run kafkacat

    ```bash
    kafkacat -C -b kafka-0.kafka-headless.kafka.svc.cluster.local:29092 -t topic
    ```

> If you don't get the expected result you can find help in the [troubleshooting section]({{< relref "docs/one-eye/logging-operator/operation/troubleshooting/_index.md">}}).
