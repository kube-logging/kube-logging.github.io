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

In this demo we are using our [Kafka operator](/docs/supertubes/kafka-operator/). For details on installing it, see the [Kafka operator installation guide](/docs/supertubes/kafka-operator/install-kafka-operator/).

## Deploy the Logging operator and a demo Application

Install the Logging operator and a demo application to provide sample log messages.

### Deploy the Logging operator with Helm {#helm}

{{< include-headless "deploy-helm-intro.md" "one-eye/logging-operator" >}}

1. Add the chart repository of the Logging operator using the following commands:

    ```bash
    helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com
    helm repo update
    ```

1. Install the Logging operator into the *logging* namespace:

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-operator banzaicloud-stable/logging-operator
    ```

1. Install the demo application and its logging definition.

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-demo banzaicloud-stable/logging-demo \
      --set "kafka.enabled=True"
    ```

1. [Validate your deployment](#validate).

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

1. [Validate your deployment](#validate).

## Validate the deployment {#validate}

Run the following command to consume some log messages from Kafka:

```bash
kubectl -n kafka run kafka-consumer -it --image=banzaicloud/kafka:2.13-2.4.0 --rm=true --restart=Never -- /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka-headless:29092 --topic topic --from-beginning
```

Expected output:

```bash
{"stream":"stdout","logtag":"F","kubernetes":{"pod_name":"logging-demo-log-generator-5f9f9cdb9f-z76wr","namespace_name":"logging","pod_id":"a7174256-31bf-4ace-897b-77899873d9ad","labels":{"app.kubernetes.io/instance":"logging-demo","app.kubernetes.io/name":"log-generator","pod-template-hash":"5f9f9cdb9f"},"host":"ip-192-168-3-189.eu-west-2.compute.internal","container_name":"log-generator","docker_id":"7349e6bb2926b8c93cb054a60f171a3f2dd1f6751c07dd389da7f28daf4d70c5","container_hash":"ghcr.io/banzaicloud/log-generator@sha256:814a69be8ab8a67aa6b009d83f6fa6c4776beefbe629a869ff16690fde8ac362","container_image":"ghcr.io/banzaicloud/log-generator:0.3.3"},"remote":"79.104.42.168","host":"-","user":"-","method":"PUT","path":"/products","code":"302","size":"18136","referer":"-","agent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.166 Safari/537.36 OPR/20.0.1396.73172","http_x_forwarded_for":"-"}
{"stream":"stdout","logtag":"F","kubernetes":{"pod_name":"logging-demo-log-generator-5f9f9cdb9f-mpp98","namespace_name":"logging","pod_id":"e2822c26-961c-4be8-99a2-b17517494ca1","labels":{"app.kubernetes.io/instance":"logging-demo","app.kubernetes.io/name":"log-generator","pod-template-hash":"5f9f9cdb9f"},"host":"ip-192-168-2-102.eu-west-2.compute.internal","container_name":"log-generator","docker_id":"26ffbec769e52e468216fe43a331f4ce5374075f9b2717d9b9ae0a7f0747b3e2","container_hash":"ghcr.io/banzaicloud/log-generator@sha256:814a69be8ab8a67aa6b009d83f6fa6c4776beefbe629a869ff16690fde8ac362","container_image":"ghcr.io/banzaicloud/log-generator:0.3.3"},"remote":"26.220.126.5","host":"-","user":"-","method":"POST","path":"/","code":"200","size":"14370","referer":"-","agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:52.0) Gecko/20100101 Firefox/52.0","http_x_forwarded_for":"-"}
```

{{< include-headless "note-troubleshooting.md" "one-eye/logging-operator" >}}
