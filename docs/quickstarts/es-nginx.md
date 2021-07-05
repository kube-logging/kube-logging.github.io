---
title: Store NGINX access logs in Elasticsearch with Logging operator
shorttitle: Elasticsearch
weight: 300
---

<p align="center"><img src="../../img/nle.png" width="340"></p>

This guide describes how to collect application and container logs in Kubernetes using the Logging operator, and how to send them to Elasticsearch.

{{< include-headless "quickstart-figure-intro.md" "one-eye/logging-operator" >}}

<p align="center"><img src="../../img/nginx-elastic.png" width="900"></p>

## Deploy Elasticsearch

First, deploy Elasticsearch in your Kubernetes cluster. The following procedure is based on the [Elastic Cloud on Kubernetes quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html), but there are some minor configuration changes, and we install everything into the *logging* namespace.

1. Install the Elasticsearch operator.

    ```yaml
    kubectl apply -f https://download.elastic.co/downloads/eck/1.3.0/all-in-one.yaml
    ```
  
1. Create the `logging` Namespace.

    ```bash
    kubectl create ns logging
    ```

1. Install the Elasticsearch cluster into the *logging* namespace.

    ```yaml
    cat <<EOF | kubectl apply -n logging -f -
    apiVersion: elasticsearch.k8s.elastic.co/v1
    kind: Elasticsearch
    metadata:
      name: quickstart
    spec:
      version: 7.10.0
      nodeSets:
      - name: default
        count: 1
        config:
          node.master: true
          node.data: true
          node.ingest: true
          node.store.allow_mmap: false
    EOF
    ```

1. Install Kibana into the *logging* namespace.

    ```yaml
    cat <<EOF | kubectl apply -n logging -f -
    apiVersion: kibana.k8s.elastic.co/v1
    kind: Kibana
    metadata:
      name: quickstart
    spec:
      version: 7.10.0
      count: 1
      elasticsearchRef:
        name: quickstart
    EOF
    ```

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
      --set "elasticsearch.enabled=True"
    ```

1. [Validate your deployment](#validate).

1. Create the `logging` resource.

     ```bash
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

1. Create an Elasticsearch `output` definition.

     ```bash
     kubectl -n logging apply -f - <<"EOF" 
     apiVersion: logging.banzaicloud.io/v1beta1
     kind: Output
     metadata:
       name: es-output
     spec:
       elasticsearch:
         host: quickstart-es-http.logging.svc.cluster.local
         port: 9200
         scheme: https
         ssl_verify: false
         ssl_version: TLSv1_2
         user: elastic
         password:
           valueFrom:
             secretKeyRef:
               name: quickstart-es-elastic-user
               key: elastic
         buffer:
           timekey: 1m
           timekey_wait: 30s
           timekey_use_utc: true
     EOF
     ```

     > Note: In production environment, use a longer `timekey` interval to avoid generating too many objects.

1. Create a `flow` resource.

     ```bash
     kubectl -n logging apply -f - <<"EOF" 
     apiVersion: logging.banzaicloud.io/v1beta1
     kind: Flow
     metadata:
       name: es-flow
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
         - es-output
     EOF
     ```

1. Install the demo application.

     ```bash
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

To validate that the deployment was successful, complete the following steps.

1. Use the following command to retrieve the password of the `elastic` user:

    ```bash
    kubectl -n logging get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
    ```

1. Enable port forwarding to the Kibana Dashboard Service.

    ```bash
    kubectl -n logging port-forward svc/quickstart-kb-http 5601
    ```

1. Open the Kibana dashboard in your browser at [https://localhost:5601](https://localhost:5601) and login as **elastic** using the retrieved password.

1. By default, the Logging operator sends the incoming log messages into an index called *fluentd*. Create an Index Pattern that includes this index (for example, *fluentd\**), then select **Menu > Kibana > Discover**. You should see the dashboard and some sample log messages from the demo application.

<p align="center"><img src="../../img/es_kibana.png" width="660"></p>

{{< include-headless "note-troubleshooting.md" "one-eye/logging-operator" >}}
