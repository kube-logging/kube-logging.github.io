---
title: AxoSyslog
weight: 200
generated_file: true
---

AxoSyslog is the schema for the AxoSyslogs API.

Available in Logging operator version 5.4 and later.

###  (metav1.TypeMeta, required) {#axosyslog-}


### metadata (metav1.ObjectMeta, optional) {#axosyslog-metadata}


### spec (AxoSyslogSpec, optional) {#axosyslog-spec}


### status (AxoSyslogStatus, optional) {#axosyslog-status}



## AxoSyslogSpec

AxoSyslogSpec defines the desired state of AxoSyslog.

### destinations ([]Destination, optional) {#axosyslogspec-destinations}

Destinations is a list of destinations to be rendered in the AxoSyslog configuration. 


### logPaths ([]LogPath, optional) {#axosyslogspec-logpaths}

LogPaths is a list of log paths to be rendered in the AxoSyslog configuration. 



## LogPath

LogPath defines a single log path that will be rendered in the AxoSyslog configuration.

### destination (string, optional) {#logpath-destination}

Name of a destination to be used in the log path. 


### filterx (string, optional) {#logpath-filterx}

FilterX block to be rendered within the log path. 



## Destination

Destination defines a single destination that will be rendered in the AxoSyslog configuration.

### config (string, optional) {#destination-config}

Config is the configuration for the destination. 


### name (string, optional) {#destination-name}

Name of the destination. 



## AxoSyslogStatus

AxoSyslogStatus defines the observed state of AxoSyslog.

### problems ([]string, optional) {#axosyslogstatus-problems}

Problems with the AxoSyslog resource. 


### problemsCount (int, optional) {#axosyslogstatus-problemscount}

Number of problems with the AxoSyslog resource. 


### sources ([]Source, optional) {#axosyslogstatus-sources}

Sources configured for AxoSyslog. 



## Source

Source represents the log sources for AxoSyslog.

### otlp (*OTLPSource, optional) {#source-otlp}

OTLP specific configuration. 



## OTLPSource

OTLPSource contains configuration for OpenTelemetry Protocol sources.

### endpoint (string, optional) {#otlpsource-endpoint}

Endpoint for the OTLP source. 



## AxoSyslogList

AxoSyslogList contains a list of AxoSyslog

###  (metav1.TypeMeta, required) {#axosysloglist-}


### metadata (metav1.ListMeta, optional) {#axosysloglist-metadata}


### items ([]AxoSyslog, required) {#axosysloglist-items}



