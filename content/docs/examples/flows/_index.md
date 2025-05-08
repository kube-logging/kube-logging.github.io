---
title: Advanced Flow examples
weight: 100
---

## CoreDNS

This Flow:

- selects [CoreDNS](https://coredns.io/) messages (every message with the `k8s-app: coredns` label),
- parses them, and
- sets a number of related [Elastic Common Schema (ECS)](https://www.elastic.co/guide/en/ecs/current/ecs-getting-started.html) fields based on the content of the message using the {{% xref "/docs/configuration/plugins/filters/record_modifier.md" %}} plugin.

{{< include-code "logging_flow_coredns.yaml" "yaml" >}}

## NGINX Ingress Controller

This Flow:

- selects [NGINX Ingress Controller](https://docs.nginx.com/nginx-ingress-controller/) messages (every message with the `app-kubernetes-io/name: ingress-nginx` label),
- parses them, and
- sets a number of related [Elastic Common Schema (ECS)](https://www.elastic.co/guide/en/ecs/current/ecs-getting-started.html) fields based on the content of the message using the {{% xref "/docs/configuration/plugins/filters/record_modifier.md" %}} plugin.
- It also adds GeoIP-related fields based on the source of the traffic using the [Fluentd GeoIP filter]({{< relref "/docs/configuration/plugins/filters/geoip.md" >}}).

{{< include-code "logging_flow_nginx_ingress.yaml" "yaml" >}}
