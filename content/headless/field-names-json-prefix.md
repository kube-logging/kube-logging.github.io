---
---
{{< warning >}}You need to use the `json.` prefix in field names.{{< /warning >}}

You can reference fields using the *dot notation*. For example, if the log contains `{"kubernetes": {"namespace_name": "default"}}`, then you can reference the `namespace_name` field using `json.kubernetes.namespace_name`.