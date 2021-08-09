---
title: FluentdSpec
weight: 200
generated_file: true
---

## FluentdSpec

FluentdSpec defines the desired state of Fluentd

### annotations (map[string]string, optional) {#fluentdspec-annotations}

Default: -

### configCheckAnnotations (map[string]string, optional) {#fluentdspec-configcheckannotations}

Default: -

### labels (map[string]string, optional) {#fluentdspec-labels}

Default: -

### envVars ([]corev1.EnvVar, optional) {#fluentdspec-envvars}

Default: -

### tls (FluentdTLS, optional) {#fluentdspec-tls}

Default: -

### image (ImageSpec, optional) {#fluentdspec-image}

Default: -

### disablePvc (bool, optional) {#fluentdspec-disablepvc}

Default: -

### bufferStorageVolume (volume.KubernetesVolume, optional) {#fluentdspec-bufferstoragevolume}

BufferStorageVolume is by default configured as PVC using FluentdPvcSpec<br>[volume.KubernetesVolume](https://github.com/banzaicloud/operator-tools/tree/master/docs/types)<br>

Default: -

### fluentdPvcSpec (*volume.KubernetesVolume, optional) {#fluentdspec-fluentdpvcspec}

Deprecated, use bufferStorageVolume<br>

Default: -

### volumeMountChmod (bool, optional) {#fluentdspec-volumemountchmod}

Default: -

### volumeModImage (ImageSpec, optional) {#fluentdspec-volumemodimage}

Default: -

### configReloaderImage (ImageSpec, optional) {#fluentdspec-configreloaderimage}

Default: -

### resources (corev1.ResourceRequirements, optional) {#fluentdspec-resources}

Default: -

### configCheckResources (corev1.ResourceRequirements, optional) {#fluentdspec-configcheckresources}

Default: -

### configReloaderResources (corev1.ResourceRequirements, optional) {#fluentdspec-configreloaderresources}

Default: -

### livenessProbe (*corev1.Probe, optional) {#fluentdspec-livenessprobe}

Default: -

### livenessDefaultCheck (bool, optional) {#fluentdspec-livenessdefaultcheck}

Default: -

### readinessProbe (*corev1.Probe, optional) {#fluentdspec-readinessprobe}

Default: -

### readinessDefaultCheck (ReadinessDefaultCheck, optional) {#fluentdspec-readinessdefaultcheck}

Default: -

### port (int32, optional) {#fluentdspec-port}

Default: -

### tolerations ([]corev1.Toleration, optional) {#fluentdspec-tolerations}

Default: -

### nodeSelector (map[string]string, optional) {#fluentdspec-nodeselector}

Default: -

### affinity (*corev1.Affinity, optional) {#fluentdspec-affinity}

Default: -

### metrics (*Metrics, optional) {#fluentdspec-metrics}

Default: -

### bufferVolumeMetrics (*Metrics, optional) {#fluentdspec-buffervolumemetrics}

Default: -

### bufferVolumeImage (ImageSpec, optional) {#fluentdspec-buffervolumeimage}

Default: -

### bufferVolumeArgs ([]string, optional) {#fluentdspec-buffervolumeargs}

Default: -

### security (*Security, optional) {#fluentdspec-security}

Default: -

### scaling (*FluentdScaling, optional) {#fluentdspec-scaling}

Default: -

### workers (int32, optional) {#fluentdspec-workers}

Default: -

### rootDir (string, optional) {#fluentdspec-rootdir}

Default: -

### logLevel (string, optional) {#fluentdspec-loglevel}

Default: -

### ignoreSameLogInterval (string, optional) {#fluentdspec-ignoresameloginterval}

Ignore same log lines<br>[more info]( https://docs.fluentd.org/deployment/logging#ignore_same_log_interval)<br>

Default: -

### ignoreRepeatedLogInterval (string, optional) {#fluentdspec-ignorerepeatedloginterval}

Ignore repeated log lines<br>[more info]( https://docs.fluentd.org/deployment/logging#ignore_repeated_log_interval)<br>

Default: -

### podPriorityClassName (string, optional) {#fluentdspec-podpriorityclassname}

Default: -

### fluentLogDestination (string, optional) {#fluentdspec-fluentlogdestination}

Default: -

### fluentOutLogrotate (*FluentOutLogrotate, optional) {#fluentdspec-fluentoutlogrotate}

FluentOutLogrotate sends fluent's stdout to file and rotates it<br>

Default: -

### forwardInputConfig (*input.ForwardInputConfig, optional) {#fluentdspec-forwardinputconfig}

Default: -

### serviceAccount (*typeoverride.ServiceAccount, optional) {#fluentdspec-serviceaccount}

Default: -

### dnsPolicy (corev1.DNSPolicy, optional) {#fluentdspec-dnspolicy}

Default: -

### dnsConfig (*corev1.PodDNSConfig, optional) {#fluentdspec-dnsconfig}

Default: -


## FluentOutLogrotate

### enabled (bool, required) {#fluentoutlogrotate-enabled}

Default: -

### path (string, optional) {#fluentoutlogrotate-path}

Default: -

### age (string, optional) {#fluentoutlogrotate-age}

Default: -

### size (string, optional) {#fluentoutlogrotate-size}

Default: -


## FluentdScaling

FluentdScaling enables configuring the scaling behaviour of the fluentd statefulset

### replicas (int, optional) {#fluentdscaling-replicas}

Default: -

### podManagementPolicy (string, optional) {#fluentdscaling-podmanagementpolicy}

Default: -

### drain (FluentdDrainConfig, optional) {#fluentdscaling-drain}

Default: -


## FluentdTLS

FluentdTLS defines the TLS configs

### enabled (bool, required) {#fluentdtls-enabled}

Default: -

### secretName (string, optional) {#fluentdtls-secretname}

Default: -

### sharedKey (string, optional) {#fluentdtls-sharedkey}

Default: -


## FluentdDrainConfig

FluentdDrainConfig enables configuring the drain behavior when scaling down the fluentd statefulset

### enabled (bool, optional) {#fluentddrainconfig-enabled}

Should buffers on persistent volumes left after scaling down the statefulset be drained<br>

Default: -

### image (ImageSpec, optional) {#fluentddrainconfig-image}

Container image to use for the drain watch sidecar<br>

Default: -


