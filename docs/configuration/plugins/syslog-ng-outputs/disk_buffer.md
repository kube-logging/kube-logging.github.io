---
title: Disk buffer
weight: 200
generated_file: true
---

## DiskBuffer

The parameters of the syslog-ng disk buffer. Using a disk buffer on the output helps avoid message loss in case of a system failure on the destination side.
Documentation: https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.37/administration-guide/56#TOPIC-1829124

### disk_buf_size (int64, required) {#diskbuffer-disk_buf_size}

Default: -

### reliable (bool, required) {#diskbuffer-reliable}

Default: -

### compaction (*bool, optional) {#diskbuffer-compaction}

Default: -

### dir (string, optional) {#diskbuffer-dir}

Default: -

### mem_buf_length (*int64, optional) {#diskbuffer-mem_buf_length}

Default: -

### mem_buf_size (*int64, optional) {#diskbuffer-mem_buf_size}

Default: -

### q_out_size (*int64, optional) {#diskbuffer-q_out_size}

Default: -


