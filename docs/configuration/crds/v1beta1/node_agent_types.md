### NodeAgent
| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| name | string | No | - | NodeAgent unique name.<br> |
| profile | string | No | linux | Specify the Logging-Operator nodeAgents profile. It can be linux or windows . <br> |
| metadata | types.MetaBase | No | - |  |
| nodeAgentFluentbit | *NodeAgentFluentbit | No | - |  |
### NodeAgentFluentbit
| Variable Name | Type | Required | Default | Description |
|---|---|---|---|---|
| enabled | *bool | No | - |  |
| daemonSet | *typeoverride.DaemonSet | No | - |  |
| serviceAccount | *typeoverride.ServiceAccount | No | - |  |
| tls | *FluentbitTLS | No | - |  |
| targetHost | string | No | - |  |
| targetPort | int32 | No | - |  |
| flush | int32 | No | 1 | Set the flush time in seconds.nanoseconds. The engine loop uses a Flush timeout to define when is required to flush the records ingested by input plugins through the defined output plugins. (default: 1)<br> |
| grace | int32 | No | 5 | Set the grace time in seconds as Integer value. The engine loop uses a Grace timeout to define wait time on exit (default: 5)<br> |
| logLevel | string | No | info | Set the logging verbosity level. Allowed values are: error, warn, info, debug and trace. Values are accumulative, e.g: if 'debug' is set, it will include error, warning, info and debug.  Note that trace mode is only available if Fluent Bit was built with the WITH_TRACE option enabled. (default: info)<br> |
| coroStackSize | int32 | No | 24576 | Set the coroutines stack size in bytes. The value must be greater than the page size of the running system. Don't set too small value (say 4096), or coroutine threads can overrun the stack buffer.<br>Do not change the default value of this parameter unless you know what you are doing. (default: 24576)<br> |
| metrics | *Metrics | No | - |  |
| metricsService | *typeoverride.Service | No | - |  |
| security | *Security | No | - |  |
| positiondb | volume.KubernetesVolume | No | - | [volume.KubernetesVolume](https://github.com/banzaicloud/operator-tools/tree/master/docs/types)<br> |
| containersPath | string | No | - |  |
| varLogsPath | string | No | - |  |
| extraVolumeMounts | []*VolumeMount | No | - |  |
| inputTail | InputTail | No | - |  |
| filterAws | *FilterAws | No | - |  |
| filterKubernetes | FilterKubernetes | No | - |  |
| disableKubernetesFilter | *bool | No | - |  |
| bufferStorage | BufferStorage | No | - |  |
| bufferStorageVolume | volume.KubernetesVolume | No | - | [volume.KubernetesVolume](https://github.com/banzaicloud/operator-tools/tree/master/docs/types)<br> |
| customConfigSecret | string | No | - |  |
| podPriorityClassName | string | No | - |  |
| livenessDefaultCheck | *bool | No | true |  |
| network | *FluentbitNetwork | No | - |  |
| forwardOptions | *ForwardOptions | No | - |  |
| enableUpstream | *bool | No | - |  |
