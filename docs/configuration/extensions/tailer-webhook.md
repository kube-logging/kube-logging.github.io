---
title: File Tailer Webhook
shorttitle: File Tailer Webhook
weight: 1000
---

Another way to keep your custom file's content tailed aside of [`host file tailer`]({{< relref "/docs/logging-extensions/reference/hosttailer_types.md" >}}) service, to configure and use the `file tailer webhook` service.
While the containers of the `host file tailers` run in a separated pod, `file tailer webhook` uses a different approach, it injects a sidecar container for every tailed file into your pod, triggered by a simple pod annotation.

## Triggering the webhook

`File tailer webhook` is based on a [`Mutating Admission Webhook`](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) which gets called every time when a pod starts, and will be triggered when a pod specification contains an annotation with the `sidecar.logging-extensions.banzaicloud.io/tail` key. For example:

```bash
apiVersion: v1
kind: Pod
metadata:
    name: test-pod
    annotations: {"sidecar.logging-extensions.banzaicloud.io/tail": "/var/log/date"}
spec:
    containers:
    - image: debian
        name: sample-container
        command: ["/bin/sh", "-c"]
        args:
            - while true; do
                date >> /var/log/date;
                sleep 1;
        done
    - image: debian
        name: sample-container2
...
```

> Note: if the pod contains multiple containers, see [Multi-container pods](#multi-container-pods).

### About the File Tailer Webhook annotation

The basic format of a `file tailer webhook` annotation is the following:

|||
|---|---|
| Key | `sidecar.logging-extensions.banzaicloud.io/tail` |
| Value | Files to be tailed separated by commas |

For example:

```bash
...
metadata:
    name: test-pod
    annotations: {"sidecar.logging-extensions.banzaicloud.io/tail": "/var/log/date,/var/log/mycustomfile"}
spec:
...
```

### Multi-container pods

In some cases you have multiple containers in your pod and you want to distinguish which file annotation belongs to which container. You can order every file annotations to particular container by prefixing the annotation with a `${ContainerName}:` container key. For example:

```bash
...
metadata:
    name: test-pod
    annotations: {"sidecar.logging-extensions.banzaicloud.io/tail": "sample-container:/var/log/date,sample-container2:/var/log/anotherfile,/var/log/mycustomfile,foobarbaz:/foo/bar/baz"}
spec:
...
```

{{< warning >}}
- Annotations without containername prefix: the file gets tailed on the default container (container 0)
- Annotations with invalid containername: file tailer annotation gets discarded
{{< /warning >}}

| Annotation | Explanation |
|---|---|
| sample-container:/var/log/date | tails file /var/log/date in sample-container |
| sample-container2:/var/log/anotherfile |  tails file /var/log/anotherfile in sample-container2 |
| /var/log/mycustomfile | tails file /var/log/mycustomfile in default container (sample-container) |
| foobarbaz:/foo/bar/baz | will be discarded due to non-existing container name |
