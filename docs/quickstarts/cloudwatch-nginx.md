---
title: Store Nginx Access Logs in Amazon CloudWatch with Logging Operator
shorttitle: Amazon CloudWatch
weight: 100
---

<p align="center"><img src="../../img/nlw.png" width="340"></p>

This guide describes how to collect application and container logs in Kubernetes using the Logging operator, and how to send them to CloudWatch.

{{< include-headless "quickstart-figure-intro.md" "one-eye/logging-operator" >}}

<p align="center"><img src="../../img/nginx-cloudwatch.png" width="900"></p>

## Deploy the Logging operator and a demo Application

Install the Logging operator and a demo application using [Helm](#helm).

### Deploy the Logging operator with Helm {#helm}

{{< include-headless "deploy-helm-intro.md" "one-eye/logging-operator" >}}

1. Add the chart repository of the Logging operator using the following commands:

    ```bash
    helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com
    helm repo update
    ```

1. Install the Logging operator.

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-operator banzaicloud-stable/logging-operator
    ```

1. Install the demo application and its logging definition.

    ```bash
   helm upgrade --install --create-namespace --namespace logging logging-demo banzaicloud-stable/logging-demo \
     --set "cloudwatch.enabled=True" \
     --set "cloudwatch.aws.secret_key=" \
     --set "cloudwatch.aws.access_key=" \
     --set "cloudwatch.aws.region=" \
     --set "cloudwatch.aws.log_group_name=" \
     --set "cloudwatch.aws.log_stream_name=" 
    ```

1. [Validate your deployment](#validate).

1. Create logging `Namespace`

    ```bash
    kubectl create ns logging
    ```

1. Create AWS `secret`

    > If you have your `$AWS_ACCESS_KEY_ID` and `$AWS_SECRET_ACCESS_KEY` set you can use the following snippet.

    ```bash
        kubectl -n logging create secret generic logging-cloudwatch --from-literal "awsAccessKeyId=$AWS_ACCESS_KEY_ID" --from-literal "awsSecretAccessKey=$AWS_SECRET_ACCESS_KEY"
    ```

    Or set up the secret manually.

    ```bash
        kubectl -n logging apply -f - <<"EOF" 
        apiVersion: v1
        kind: Secret
        metadata:
          name: logging-cloudwatch
        type: Opaque
        data:
          awsAccessKeyId: <base64encoded>
          awsSecretAccessKey: <base64encoded>
        EOF
    ```

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

1. Create an CloudWatch `output` definition.

     ```bash
    kubectl -n logging apply -f - <<"EOF" 
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Output
    metadata:
      name: cloudwatch-output
      namespace: logging
    spec:
      cloudwatch:
        aws_key_id:
          valueFrom:
            secretKeyRef:
              name: logging-cloudwatch
              key: awsAccessKeyId
        aws_sec_key:
          valueFrom:
            secretKeyRef:
              name: logging-cloudwatch
              key: awsSecretAccessKey
        log_group_name: operator-log-group
        log_stream_name: operator-log-stream
        region: us-east-1
        auto_create_stream: true
        buffer:
          timekey: 30s
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
       name: cloudwatch-flow
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
         - cloudwatch-output
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

<p align="center"><img src="../../img/cw.png" width="660"></p>

{{< include-headless "note-troubleshooting.md" "one-eye/logging-operator" >}}
