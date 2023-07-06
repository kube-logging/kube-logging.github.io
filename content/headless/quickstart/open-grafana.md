---
---
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

