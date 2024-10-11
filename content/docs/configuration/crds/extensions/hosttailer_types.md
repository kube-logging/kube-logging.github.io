---
title: HostTailer
weight: 200
generated_file: true
---

## HostTailerSpec

HostTailerSpec defines the desired state of HostTailer

### enableRecreateWorkloadOnImmutableFieldChange (bool, optional) {#hosttailerspec-enablerecreateworkloadonimmutablefieldchange}

EnableRecreateWorkloadOnImmutableFieldChange enables the operator to recreate the daemonset (and possibly other resource in the future) in case there is a change in an immutable field that otherwise couldn't be managed with a simple update. 


### fileTailers ([]FileTailer, optional) {#hosttailerspec-filetailers}

List of [file tailers](#filetailer).


### image (tailer.ImageSpec, optional) {#hosttailerspec-image}


### systemdTailers ([]SystemdTailer, optional) {#hosttailerspec-systemdtailers}

List of [systemd tailers](#systemdtailer).


### workloadOverrides (*types.PodSpecBase, optional) {#hosttailerspec-workloadoverrides}

Override podSpec fields for the given daemonset 


### workloadMetaOverrides (*types.MetaBase, optional) {#hosttailerspec-workloadmetaoverrides}

Override metadata of the created resources 



## HostTailerStatus

HostTailerStatus defines the observed state of [HostTailer](#hosttailer).


## HostTailer

HostTailer is the Schema for the hosttailers API

###  (metav1.TypeMeta, required) {#hosttailer-}


### metadata (metav1.ObjectMeta, optional) {#hosttailer-metadata}


### spec (HostTailerSpec, optional) {#hosttailer-spec}


### status (HostTailerStatus, optional) {#hosttailer-status}



## HostTailerList

HostTailerList contains a list of [HostTailers](#hosttailer).

###  (metav1.TypeMeta, required) {#hosttailerlist-}


### metadata (metav1.ListMeta, optional) {#hosttailerlist-metadata}


### items ([]HostTailer, required) {#hosttailerlist-items}



## FileTailer

FileTailer configuration options

### buffer_chunk_size (string, optional) {#filetailer-buffer_chunk_size}

Set the buffer chunk size per active filetailer 


### buffer_max_size (string, optional) {#filetailer-buffer_max_size}

Set the limit of the buffer size per active filetailer 


### containerOverrides (*types.ContainerBase, optional) {#filetailer-containeroverrides}

Override container fields for the given tailer 


### disabled (bool, optional) {#filetailer-disabled}

Disable tailing the file 


### image (*tailer.ImageSpec, optional) {#filetailer-image}

Override image field for the given trailer 


### name (string, required) {#filetailer-name}

Name for the tailer 


### path (string, optional) {#filetailer-path}

Path to the loggable file 


### read_from_head (bool, optional) {#filetailer-read_from_head}

Start reading from the head of new log files 


### skip_long_lines (string, optional) {#filetailer-skip_long_lines}

Skip long line when exceeding Buffer_Max_Size 



## SystemdTailer

SystemdTailer configuration options

### containerOverrides (*types.ContainerBase, optional) {#systemdtailer-containeroverrides}

Override container fields for the given tailer 


### disabled (bool, optional) {#systemdtailer-disabled}

Disable component 


### image (*tailer.ImageSpec, optional) {#systemdtailer-image}

Override image field for the given trailer 


### maxEntries (int, optional) {#systemdtailer-maxentries}

Maximum entries to read when starting to tail logs to avoid high pressure 


### name (string, required) {#systemdtailer-name}

Name for the tailer 


### path (string, optional) {#systemdtailer-path}

Override systemd log path 


### systemdFilter (string, optional) {#systemdtailer-systemdfilter}

Filter to select the systemd unit, for example: `kubelet.service`
If the `systemdFilter` is not specified, `_SYSTEMD_UNIT` is used.

For example, the following HostTailer creates a systemd tailer for the `kubelet.service` and for kernel logs, and logs the kernel logs under the `SYSLOG_IDENTIFIER` journal field.

{{< highlight yaml >}}
apiVersion: logging-extensions.banzaicloud.io/v1alpha1
kind: HostTailer
metadata:
  labels:
    app.kubernetes.io/name: systemd-hosttailer
  name: systemd
  namespace: logging
spec:
  systemdTailers:
  - maxEntries: 100
    name: kubelet
    systemdFilter: kubelet.service
  - maxEntries: 100
    name: kernel
    systemdFilter: SYSLOG_IDENTIFIER=kernel
{{</ highlight >}}

The generated container will have the following arguments:

{{< highlight yaml >}}
      - command:
        - /fluent-bit/bin/fluent-bit
        - -i
        - systemd
        - -p
        - path=/var/log/journal
        - -p
        - db=/var/pos/systemd-host-tailer-kernel.db
        - -p
        - max_entries=100
        - -p
        - systemd_filter=SYSLOG_IDENTIFIER=kernel
        - -o
        - file
        - -p
        - format=plain
        - -p
        - path=/dev/
        - -p
        - file=stdout
{{</ highlight >}}
