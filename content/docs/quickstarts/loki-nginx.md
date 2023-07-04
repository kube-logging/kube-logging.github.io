---
title: Logging operator with Fluentd
weight: 200
aliases:
    - /docs/examples/loki-nginx/
---

This guide shows you how to collect application and container logs in Kubernetes using the Logging operator. As the Logging operator itself doesn't store any logs, you will install a [Grafana Loki](https://grafana.com/docs/loki/latest/) instance and configure the Logging operator to send your log messages to Loki for short-term storage.

{{< include-headless "quickstart-figure-intro.md" >}}

<p align="center"><img src="../../img/nginx-loki.png" width="900"></p>

## Deploy Loki and Grafana

First, deploy Loki and Grafana to your cluster. Loki will store the collected logs, and you can browse the logs using the Grafana dashboard.

1. Add the chart repositories of Loki and Grafana using the following commands:

    ```bash
    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo add loki https://grafana.github.io/loki/charts
    helm repo update
    ```

1. Install Loki into the *logging* namespace:

    ```bash
    helm upgrade --install --create-namespace --namespace logging loki loki/loki
    ```

    > Note: For details on installing Loki, see the [official Grafana Loki Documentation](https://grafana.com/docs/loki/latest/installation/helm/).

1. Install Grafana into the *logging* namespace:

   ```bash
    helm upgrade --install --create-namespace --namespace logging grafana grafana/grafana \
    --set "datasources.datasources\\.yaml.apiVersion=1" \
    --set "datasources.datasources\\.yaml.datasources[0].name=Loki" \
    --set "datasources.datasources\\.yaml.datasources[0].type=loki" \
    --set "datasources.datasources\\.yaml.datasources[0].url=http://loki:3100" \
    --set "datasources.datasources\\.yaml.datasources[0].access=proxy"
    ```

## Deploy the Logging operator with Helm {#helm}

Install the Logging operator and a log-generator application to create sample log messages.

{{< include-headless "deploy-helm-intro.md" >}}

1. Add the chart repository of the Logging operator using the following commands:

    ```bash
    helm repo add kube-logging https://kube-logging.dev/helm-charts
    helm repo update
    ```

1. Install the Logging operator into the *logging* namespace:

    ```bash
    helm upgrade --install --wait --create-namespace --namespace logging logging-operator kube-logging/logging-operator
    ```

## Configure the Logging operator

1. Create a `Logging` resource to deploy the Fluent Bit log collector agent on the nodes of the cluster, and Fluentd as the central log forwarder.

     ```bash
     kubectl --namespace logging apply -f - <<"EOF"
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

     > Note: You can use `ClusterOutput` and `ClusterFlow` resources only in the `controlNamespace`. For details, see {{% xref "/docs/configuration/_index.md" %}}.

1. Create an `Output` resource (called `loki-output` in the example) that configures a Loki output.

     ```bash
    kubectl --namespace logging apply -f - <<"EOF"
    apiVersion: logging.banzaicloud.io/v1beta1
    kind: Output
    metadata:
      name: loki-output
    spec:
      loki:
        url: http://loki:3100
        configure_kubernetes_labels: true
        buffer:
          timekey: 1m
          timekey_wait: 30s
          timekey_use_utc: true
    EOF
     ```

     > Note: In production environment, use a longer `timekey` interval to avoid generating too many objects.

1. Create a `Flow` resource that routes the collected logs to the Loki output.

    <!-- FIXME simplify the example if possible -->

     ```bash
     kubectl --namespace logging apply -f - <<"EOF"
     apiVersion: logging.banzaicloud.io/v1beta1
     kind: Flow
     metadata:
       name: loki-flow
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
         - loki-output
     EOF
     ```

1. Install log-generator to produce logs with the label `app.kubernetes.io/name: log-generator`

     ```bash
     helm upgrade --install --wait --create-namespace --namespace logging log-generator kube-logging/log-generator
     ```

1. [Check the collected logs on the Grafana Dashboard](#grafana).

## Open Grafana Dashboard {#grafana}

Open the Grafana Dashboard and check the collected logs.

1. Use the following command to retrieve the password of the Grafana `admin` user:

    ```bash
    kubectl get secret --namespace logging grafana --output jsonpath="{.data.admin-password}" | base64 --decode ; echo
    ```

1. Enable port forwarding to the Grafana Service.

    ```bash
    kubectl --namespace logging port-forward svc/grafana 3000:80
    ```

1. Open the Grafana Dashboard: [http://localhost:3000](http://localhost:3000)

1. Use the `admin` username and the password retrieved in Step 1 to log in.

1. Select **Menu > Explore**, select **Data source > Loki**, then select **Log labels > namespace > logging**. A list of logs should appear.

    ![Sample log messages in Loki](../../img/loki1.png)

{{< include-headless "note-troubleshooting.md" >}}

<!-- FIXME add another simple usecase (filtering, or another namespace), and check the dashboard again -->

## Summary

If you have completed this guide, you have made the following changes to your cluster:

- Installed the Fluent Bit agent on every node of the cluster that collects the logs and the labels from the node.
- Installed Fluentd on the cluster, which receives the logs from the Fluent Bit agents, and can filter, parse, and transform them as needed. Fluentd also routes the incoming logs to an output. To learn more about routing and filtering, see {{% xref "/docs/configuration/log-routing.md" %}}.
- Installed [Grafana Loki](https://grafana.com/docs/loki/latest/) to store your logs, and configured Fluentd to send the incoming logs to the Loki output.
- Installed [Grafana](https://grafana.com/docs/grafana/latest/) that allows you to browse the logs stored in Loki.
- Created the following resources that configure Logging operator and the components it manages:

    - `Logging` to configure the logging infrastructure, like the details of the Fluent Bit and Fluentd deployment. To learn more about configuring the logging infrastructure, see {{% xref "/docs/logging-infrastructure/_index.md" %}}.
    - `Output` to define a Loki output that receives the collected messages. To learn more, see {{% xref "/docs/configuration/output.md" %}}.
    - `Flow` that processes the incoming messages and routes them to the appropriate output. To learn more, see {{% xref "/docs/configuration/flow.md" %}}.

{{< include-headless "support-troubleshooting.md" >}}
