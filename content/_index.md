---
title: Logging operator
---

{{< blocks/cover title="Welcome to Logging operator!" image_anchor="top" avatar_image="/icons/logo-no-black.svg" width="min" color="gray" >}}
<div class="mx-auto">
	<a class="btn btn-lg btn-primary mr-3 mb-4" href="{{< relref "/docs/" >}}">
		Learn More <i class="fa-solid fa-circle-right ml-2"></i>
	</a>
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/docs/install/">
		Install <i class="fa-brands fa-github ml-2 "></i>
	</a>
	<p class="lead mt-5">The Logging operator solves your logging-related problems in Kubernetes environments by automating the deployment and configuration of a Kubernetes logging pipeline.</p>
</div>
{{< /blocks/cover >}}

{{% blocks/lead color="primary" %}}
The Logging operator manages the log collectors and log forwarders of your logging infrastructure, and the routing rules that specify where you want to send your different log messages. You can filter and process the incoming log messages using the flow custom resource of the log forwarder to route them to the appropriate output. The outputs are the destinations where you want to send your log messages, for example, Elasticsearch, or an Amazon S3 bucket. You can also define cluster-wide outputs and flows, for example, to use a centralized output that namespaced users can reference but cannot modify.
{{% /blocks/lead %}}

{{< blocks/section color="dark" type="features">}}
{{% blocks/feature icon="fa-lightbulb" title="Learn more about Logging operator!" url="/docs/" %}}
Read the Logging operator documentation.
{{% /blocks/feature %}}


{{% blocks/feature icon="fa-brands fa-github" title="Contributions welcome!" url="https://github.com/kube-logging/logging-operator" %}}
We do a [Pull Request](https://github.com/kube-logging/logging-operator/pulls) contributions workflow on **GitHub**. New users and developers are always welcome!
{{% /blocks/feature %}}


{{% blocks/feature icon="fa-brands fa-slack" title="Come chat with us!" url="https://discord.gg/eAcqmAVU2u" url_text="Join Discord" %}}
In case you need help, you can find on <a href="https://join.slack.com/t/emergingtechcommunity/shared_invite/zt-1rw2jl0ht-yNdyFgBFlc%7Eyzo9AnE4FbA">Slack</a> and <a href="https://discord.gg/9ACY4RDsYN">Discord</a>.
{{% /blocks/feature %}}

{{< /blocks/section >}}
