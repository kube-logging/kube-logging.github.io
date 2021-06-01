---
title: Parser
weight: 200
generated_file: true
---

# [Parser Filter](https://docs.fluentd.org/filter/parser)
## Overview
 Parses a string field in event records and mutates its event record with the parsed result.

## Configuration
## ParserConfig

### key_name (string, optional) {#parserconfig-key_name}

Specify field name in the record to parse. If you leave empty the Container Runtime default will be used.<br>

Default: -

### reserve_time (bool, optional) {#parserconfig-reserve_time}

Keep original event time in parsed result.<br>

Default: -

### reserve_data (bool, optional) {#parserconfig-reserve_data}

Keep original key-value pair in parsed result.<br>

Default: -

### remove_key_name_field (bool, optional) {#parserconfig-remove_key_name_field}

Remove key_name field when parsing is succeeded<br>

Default: -

### replace_invalid_sequence (bool, optional) {#parserconfig-replace_invalid_sequence}

If true, invalid string is replaced with safe characters and re-parse it.<br>

Default: -

### inject_key_prefix (string, optional) {#parserconfig-inject_key_prefix}

Store parsed values with specified key name prefix.<br>

Default: -

### hash_value_field (string, optional) {#parserconfig-hash_value_field}

Store parsed values as a hash value in a field.<br>

Default: -

### emit_invalid_record_to_error (*bool, optional) {#parserconfig-emit_invalid_record_to_error}

Emit invalid record to @ERROR label. Invalid cases are: key not exist, format is not matched, unexpected error<br>

Default: -

### parse (ParseSection, optional) {#parserconfig-parse}

[Parse Section](#parse-section)<br>

Default: -

### parsers ([]ParseSection, optional) {#parserconfig-parsers}

Deprecated, use `parse` instead<br>

Default: -


## Parse Section

### type (string, optional) {#parse section-type}

Parse type: apache2, apache_error, nginx, syslog, csv, tsv, ltsv, json, multiline, none, logfmt<br>

Default: -

### expression (string, optional) {#parse section-expression}

Regexp expression to evaluate<br>

Default: -

### time_key (string, optional) {#parse section-time_key}

Specify time field for event time. If the event doesn't have this field, current time is used.<br>

Default: -

### null_value_pattern (string, optional) {#parse section-null_value_pattern}

Specify null value pattern.<br>

Default: -

### null_empty_string (bool, optional) {#parse section-null_empty_string}

If true, empty string field is replaced with nil<br>

Default: -

### estimate_current_event (bool, optional) {#parse section-estimate_current_event}

If true, use Fluent::EventTime.now(current time) as a timestamp when time_key is specified.<br>

Default: -

### keep_time_key (bool, optional) {#parse section-keep_time_key}

If true, keep time field in the record.<br>

Default: -

### types (string, optional) {#parse section-types}

Types casting the fields to proper types example: field1:type, field2:type<br>

Default: -

### time_format (string, optional) {#parse section-time_format}

Process value using specified format. This is available only when time_type is string<br>

Default: -

### time_type (string, optional) {#parse section-time_type}

Parse/format value according to this type available values: float, unixtime, string <br>

Default:  string

### local_time (bool, optional) {#parse section-local_time}

Ff true, use local time. Otherwise, UTC is used. This is exclusive with utc. <br>

Default:  true

### utc (bool, optional) {#parse section-utc}

If true, use UTC. Otherwise, local time is used. This is exclusive with localtime <br>

Default:  false

### timezone (string, optional) {#parse section-timezone}

Use specified timezone. one can parse/format the time value in the specified timezone. <br>

Default:  nil

### format (string, optional) {#parse section-format}

Only available when using type: multi_format<br>

Default: -

### format_firstline (string, optional) {#parse section-format_firstline}

Only available when using type: multi_format<br>

Default: -

### delimiter (string, optional) {#parse section-delimiter}

Only available when using type: ltsv <br>

Default:  "\t"

### delimiter_pattern (string, optional) {#parse section-delimiter_pattern}

Only available when using type: ltsv<br>

Default: -

### label_delimiter (string, optional) {#parse section-label_delimiter}

Only available when using type: ltsv <br>

Default:  ":"

### multiline ([]string, optional) {#parse section-multiline}

The multiline parser plugin parses multiline logs.<br>

Default: -

### patterns ([]SingleParseSection, optional) {#parse section-patterns}

Only available when using type: multi_format<br>[Parse Section](#parse-section)<br>

Default: -


## Parse Section (single)

### type (string, optional) {#parse section (single)-type}

Parse type: apache2, apache_error, nginx, syslog, csv, tsv, ltsv, json, multiline, none, logfmt<br>

Default: -

### expression (string, optional) {#parse section (single)-expression}

Regexp expression to evaluate<br>

Default: -

### time_key (string, optional) {#parse section (single)-time_key}

Specify time field for event time. If the event doesn't have this field, current time is used.<br>

Default: -

### null_value_pattern (string, optional) {#parse section (single)-null_value_pattern}

Specify null value pattern.<br>

Default: -

### null_empty_string (bool, optional) {#parse section (single)-null_empty_string}

If true, empty string field is replaced with nil<br>

Default: -

### estimate_current_event (bool, optional) {#parse section (single)-estimate_current_event}

If true, use Fluent::EventTime.now(current time) as a timestamp when time_key is specified.<br>

Default: -

### keep_time_key (bool, optional) {#parse section (single)-keep_time_key}

If true, keep time field in the record.<br>

Default: -

### types (string, optional) {#parse section (single)-types}

Types casting the fields to proper types example: field1:type, field2:type<br>

Default: -

### time_format (string, optional) {#parse section (single)-time_format}

Process value using specified format. This is available only when time_type is string<br>

Default: -

### time_type (string, optional) {#parse section (single)-time_type}

Parse/format value according to this type available values: float, unixtime, string <br>

Default:  string

### local_time (bool, optional) {#parse section (single)-local_time}

Ff true, use local time. Otherwise, UTC is used. This is exclusive with utc. <br>

Default:  true

### utc (bool, optional) {#parse section (single)-utc}

If true, use UTC. Otherwise, local time is used. This is exclusive with localtime <br>

Default:  false

### timezone (string, optional) {#parse section (single)-timezone}

Use specified timezone. one can parse/format the time value in the specified timezone. <br>

Default:  nil

### format (string, optional) {#parse section (single)-format}

Only available when using type: multi_format<br>

Default: -


 #### Example `Parser` filter configurations
 ```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: demo-flow
spec:
  filters:
    - parser:
        remove_key_name_field: true
        reserve_data: true
        parse:
          type: multi_format
          patterns:
          - format: nginx
          - format: regexp
            expression: /foo/
          - format: none
  selectors: {}
  localOutputRefs:
    - demo-output
 ```

 #### Fluentd Config Result
 ```yaml
<filter **>
  @type parser
  @id test_parser
  key_name message
  remove_key_name_field true
  reserve_data true
  <parse>
    @type multi_format
    <pattern>
      format nginx
    </pattern>
    <pattern>
      expression /foo/
      format regexp
    </pattern>
    <pattern>
      format none
    </pattern>
  </parse>
</filter>
 ```

---
