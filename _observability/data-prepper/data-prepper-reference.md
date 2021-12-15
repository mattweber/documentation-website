---
layout: default
title: Configuration reference
parent: Data Prepper
nav_order: 3
---

# Data Prepper configuration reference

This page lists all supported Data Prepper server, sources, buffers, preppers, and sinks, along with their associated options. For example configuration files, see [Data Prepper]({{site.url}}{{site.baseurl}}/observability/data-prepper/pipelines/).

## Data Prepper server options

Option | Required | Description
:--- | :--- | :---
ssl | No | Boolean, indicating whether TLS should be used for server APIs. Defaults to true.
keyStoreFilePath | No | String, path to a .jks or .p12 keystore file. Required if ssl is true.
keyStorePassword | No | String, password for keystore. Optional, defaults to empty string.
privateKeyPassword | No | String, password for private key within keystore. Optional, defaults to empty string.
serverPort | No | Integer, port number to use for server APIs. Defaults to 4900
metricRegistries | No | List, metrics registries for publishing the generated metrics. Defaults to Prometheus; Prometheus and CloudWatch are currently supported.

## General pipeline options

Option | Required | Description
:--- | :--- | :---
workers | No | Integer, default 1. Essentially the number of application threads. As a starting point for your use case, try setting this value to the number of CPU cores on the machine.
delay | No | Integer (milliseconds), default 3,000. How long workers wait between buffer read attempts.


## Sources

Sources define where your data comes from.


### otel_trace_source

Source for the OpenTelemetry Collector.

Option | Required | Description
:--- | :--- | :---
port | No | Integer, the port OTel trace source is running on. Default is `21890`.
request_timeout | No | Integer, the request timeout in millis. Default is `10_000`.
health_check_service | No | Boolean, enables a gRPC health check service under `grpc.health.v1/Health/Check`. Default is `false`.
proto_reflection_service | No | Boolean, enables a reflection service for Protobuf services (see [gRPC reflection](https://github.com/grpc/grpc/blob/master/doc/server-reflection.md) and [gRPC Server Reflection Tutorial](https://github.com/grpc/grpc-java/blob/master/documentation/server-reflection-tutorial.md) docs). Default is `false`.
unframed_requests | No | Boolean, enable requests not framed using the gRPC wire protocol.
thread_count | No | Integer, the number of threads to keep in the ScheduledThreadPool. Default is `200`.
max_connection_count | No | Integer, the maximum allowed number of open connections. Default is `500`.
ssl | No | Boolean, enables connections to the OTel source port over TLS/SSL. Defaults to `true`.
sslKeyCertChainFile | Conditionally | String, file-system path or AWS S3 path to the security certificate (e.g. `"config/demo-data-prepper.crt"` or `"s3://my-secrets-bucket/demo-data-prepper.crt"`). Required if ssl is set to `true`.
sslKeyFile | Conditionally | String, file-system path or AWS S3 path to the security key (e.g. `"config/demo-data-prepper.key"` or `"s3://my-secrets-bucket/demo-data-prepper.key"`). Required if ssl is set to `true`.
useAcmCertForSSL | No | Boolean, enables TLS/SSL using certificate and private key from AWS Certificate Manager (ACM). Default is `false`.
acmCertificateArn | Conditionally | String, represents the ACM certificate ARN. ACM certificate take preference over S3 or local file system certificate. Required if `useAcmCertForSSL` is set to `true`.
awsRegion | Conditionally | String, represents the AWS region to use ACM or S3. Required if `useAcmCertForSSL` is set to `true` or `sslKeyCertChainFile` and `sslKeyFile` are AWS S3 paths.
authentication | No | An authentication configuration. By default, this runs an unauthenticated server. This uses pluggable authentication for HTTPS. To use basic authentication define the ```http_basic``` plugin with a `username` and `password`. To provide customer authentication use or create a plugin which implements: [GrpcAuthenticationProvider](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/armeria-common/src/main/java/com/amazon/dataprepper/armeria/authentication/GrpcAuthenticationProvider.java).

### http_source

This is a source plugin that supports HTTP protocol. Currently ONLY support Json UTF-8 codec for incoming request, e.g. `[{"key1": "value1"}, {"key2": "value2"}]`.

Option | Required | Description
:--- | :--- | :---
port | No | Integer, the port the source is running on. Default is `2021`. Valid options are between `0` and `65535`.
request_timeout | No | Integer, the request timeout in millis. Default is `10_000`.
thread_count | No | Integer, the number of threads to keep in the ScheduledThreadPool. Default is `200`.
max_connection_count | No | Integer, the maximum allowed number of open connections. Default is `500`.
max_pending_requests | No | Ingeger, the maximum number of allowed tasks in ScheduledThreadPool work queue. Default is `1024`.
authentication | No | An authentication configuration. By default, this runs an unauthenticated server. This uses pluggable authentication for HTTPS. To use basic authentication define the ```http_basic``` plugin with a `username` and `password`. To provide customer authentication use or create a plugin which implements: [ArmeriaHttpAuthenticationProvider](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/armeria-common/src/main/java/com/amazon/dataprepper/armeria/authentication/ArmeriaHttpAuthenticationProvider.java).

### file

Source for flat file input.

Option | Required | Description
:--- | :--- | :---
path | Yes | String, path to the input file (e.g. `logs/my-log.log`).
format | No | String, format of each line in the file. Valid options are `json` or `plain`. Default is `plain`.
record_type | No | String, the record type that will be stored. Valid options are `string` or `event`. Default is `string`. If you would like to use the file source for log analytics use cases like grok, change this to `event`.

### pipeline

Source for reading from another pipeline.

Option | Required | Description
:--- | :--- | :---
name | Yes | String, name of the pipeline to read from.


### stdin

Source for console input. Can be useful for testing. No options.


## Buffers

Buffers store data as it passes through the pipeline. If you implement a custom buffer, it can be memory-based (better performance) or disk-based (larger).


### bounded_blocking

The default buffer. Memory-based.

Option | Required | Description
:--- | :--- | :---
buffer_size | No | Integer, default 512. The maximum number of records the buffer accepts.
batch_size | No | Integer, default 8. The maximum number of records the buffer drains after each read.


## Preppers

Preppers perform some action on your data: filter, transform, enrich, etc.


### otel_trace_raw_prepper

Converts OpenTelemetry data to OpenSearch-compatible JSON documents.

Option | Required | Description
:--- | :--- | :---
root_span_flush_delay | No | Integer, representing the time interval in seconds to flush all the root spans in the prepper together with their descendants. Defaults to 30.
trace_flush_interval | No | Integer, representing the time interval in seconds to flush all the descendant spans without any root span. Defaults to 180.


### service_map_stateful

Uses OpenTelemetry data to create a distributed service map for visualization in OpenSearch Dashboards.

Option | Required | Description
:--- | :--- | :---
window_duration | No | Integer, representing the fixed time window in seconds to evaluate service-map relationships. Defaults to 180.

### peer_forwarder

Forwards ExportTraceServiceRequests via gRPC to other Data Prepper instances. Required for operating Data Prepper in a clustered deployment.

Option | Required | Description
:--- | :--- | :---
time_out | No | Integer, forwarded request timeout in seconds. Defaults to 3 seconds.
span_agg_count | No | Integer, batch size for number of spans per request. Defaults to 48.
target_port | No | Integer, the destination port to forward requests to. Defaults to `21890`.
discovery_mode | No | String, peer discovery mode to be used. Allowable values are `static`, `dns`, and `aws_cloud_map`. Defaults to `static`.
static_endpoints | No | List, containing string endpoints of all Data Prepper instances.
domain_name | No | String, single domain name to query DNS against. Typically used by creating multiple DNS A Records for the same domain.
ssl | No | Boolean, indicating whether TLS should be used. Default is true.
awsCloudMapNamespaceName | Conditionally | String, name of your CloudMap Namespace. Required if `discovery_mode` is set to `aws_cloud_map`.
awsCloudMapServiceName | Conditionally | String, service name within your CloudMap Namespace. Required if `discovery_mode` is set to `aws_cloud_map`.
sslKeyCertChainFile | Conditionally | String, represents the SSL certificate chain file path or AWS S3 path. S3 path example `s3://<bucketName>/<path>`. Required if `ssl` is set to `true`.
useAcmCertForSSL | No | Boolean, enables TLS/SSL using certificate and private key from AWS Certificate Manager (ACM). Default is `false`.
awsRegion | Conditionally | String, represents the AWS region to use ACM, S3, or CloudMap. Required if `useAcmCertForSSL` is set to `true` or `sslKeyCertChainFile` and `sslKeyFile` are AWS S3 paths.
acmCertificateArn | Conditionally | String represents the ACM certificate ARN. ACM certificate take preference over S3 or local file system certificate. Required if `useAcmCertForSSL` is set to `true`.

### string_converter

Converts strings to uppercase or lowercase. Mostly useful as an example if you want to develop your own prepper.

Option | Required | Description
:--- | :--- | :---
upper_case | No | Boolean, whether to convert to uppercase (`true`) or lowercase (`false`).

### grok_prepper

Takes unstructured data and utilizes pattern matching to structure and extract important keys and make data more structured and queryable.

Option | Required | Description
:--- | :--- | :---
match | No | Map, specifies which keys to match specific patterns against. Default is an empty body.
keep_empty_captures | No | Boolean, enables preserving `null` captures. Default value is `false`.
named_captures_only | No | Boolean, enables whether to keep only named captures. Default value is `true`.
break_on_match | No | Boolean, specifies wether to match all patterns or stop once the first successful match is found. Default is `true`.
keys_to_overwrite | No | List, specifies which existing keys are to be overwritten if there is a capture with the same key value. Default is `[]`.
pattern_definitions | No | Map, that allows for custom pattern use inline. Default value is `{}`.
patterns_directories | No | List, specifies the path of directories that contain customer pattern files. Default value is `[]`.
pattern_files_glob | No | String, specifies which pattern files to use from the directories specified for `pattern_directories`. Default is `*`.
target_key | No | String, specifies a parent level key to store all captures. Default value is `null`.
timeout_millis | No | Integer, the maximum amount of time that matching will be performed. Setting to `0` will disable the timeout. Default value is `30,000`.

## Sinks

Sinks define where Data Prepper writes your data to.


### opensearch

Sink for an OpenSearch cluster.

Option | Required | Description
:--- | :--- | :---
hosts | Yes | List of OpenSearch hosts to write to (e.g. `["https://localhost:9200", "https://remote-cluster:9200"]`).
cert | No | String, path to the security certificate (e.g. `"config/root-ca.pem"`) if the cluster uses the OpenSearch security plugin.
username | No | String, username for HTTP basic authentication.
password | No | String, password for HTTP basic authentication.
aws_sigv4 | No | Boolean, default false. Whether to use IAM signing to connect to an Amazon OpenSearch Service domain. For your access key, secret key, and optional session token, Data Prepper uses the default credential chain (environment variables, Java system properties, `~/.aws/credential`, etc.).
aws_region | No | String, AWS region (e.g. `"us-east-1"`) for the domain if you are connecting to Amazon OpenSearch Service.
aws_sts_role_arn | No | String, IAM role which the sink plugin assumes to sign request to Amazon OpenSearch Service. If not provided the plugin uses the default credentials.
socket_timeout | No | Integer, the timeout in milliseconds for waiting for data (or, put differently, a maximum period inactivity between two consecutive data packets). A timeout value of zero is interpreted as an infinite timeout. If this timeout value is either negative or not set, the underlying Apache HttpClient would rely on operating system settings for managing socket timeouts.
connect_timeout | No | Integer, the timeout in milliseconds used when requesting a connection from the connection manager. A timeout value of zero is interpreted as an infinite timeout. If this timeout value is either negative or not set, the underlying Apache HttpClient would rely on operating system settings for managing connection timeouts.
insecure | No | Boolean, default false. Whether to verify SSL certificates. If set to true, CA certificate verification is disabled and insecure HTTP requests are sent instead.
proxy | No | String, the address of a [forward HTTP proxy server](https://en.wikipedia.org/wiki/Proxy_server). The format is like "<host name or IP>:<port>". Examples: "example.com:8100", "http://example.com:8100", "112.112.112.112:8100". Note: port number cannot be omitted.
trace_analytics_raw | No | Boolean, default false. Deprecated in favor of `index_type`. Whether to export as trace data to the `otel-v1-apm-span-*` index pattern (alias `otel-v1-apm-span`) for use with the Trace Analytics OpenSearch Dashboards plugin.
trace_analytics_service_map | No | Boolean, default false. Deprecated in favor of `index_type`. Whether to export as trace data to the `otel-v1-apm-service-map` index for use with the service map component of the Trace Analytics OpenSearch Dashboards plugin.
index | No | String, name of the index to export to. Only required if you don't use the `trace-analytics-raw` or `trace-analytics-service-map` presets. In other words, this parameter is applicable and required only if index_type is explicitly `custom` or defaults to `custom`.
index_type | No | String, default `custom`. This index type instructs the Sink plugin what type of data it is handling. Valid values: `custom`, `trace-analytics-raw`, `trace-analytics-service-map`.
template_file | No | String, the path to a JSON [index template]({{site.url}}{{site.baseurl}}/opensearch/index-templates/) file (e.g. `/your/local/template-file.json` if you do not use the `trace_analytics_raw` or `trace_analytics_service_map`. See [otel-v1-apm-span-index-template.json](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/opensearch/src/main/resources/otel-v1-apm-span-index-template.json) for an example.
document_id_field | No | String, the field from the source data to use for the OpenSearch document ID (e.g. `"my-field"`) if you don't use the `trace_analytics_raw` or `trace_analytics_service_map` presets.
dlq_file | No | String, the path to your preferred dead letter queue file (e.g. `/your/local/dlq-file`). Data Prepper writes to this file when it fails to index a document on the OpenSearch cluster.
bulk_size | No | Integer (long), default 5. The maximum size (in MiB) of bulk requests to the OpenSearch cluster. Values below 0 indicate an unlimited size. If a single document exceeds the maximum bulk request size, Data Prepper sends it individually.
ism_policy_file | No | String, the absolute file path for an ISM (Index State Management) policy JSON file. This policy file is effective only when there is no built-in policy file for the index type. For example, `custom` index type is currently the only one without a built-in policy file, thus it would use the policy file here if it's provided through this parameter. For more information, see [ISM policies](https://opensearch.org/docs/latest/im-plugin/ism/policies/).
number_of_shards | No | Integer, the number of primary shards that an index should have on the destination OpenSearch server. This parameter is effective only when `template_file` is either explicitly provided in Sink configuration or built-in. If this parameter is set, it would override the value in index template file. For more information, see [create index](https://opensearch.org/docs/latest/opensearch/rest-api/index-apis/create-index/).
number_of_replicas | No | Integer, the number of replica shards each primary shard should have on the destination OpenSearch server. For example, if you have 4 primary shards and set number_of_replicas to 3, the index has 12 replica shards. This parameter is effective only when `template_file` is either explicitly provided in Sink configuration or built-in. If this parameter is set, it would override the value in index template file. For more information, see [create index](https://opensearch.org/docs/latest/opensearch/rest-api/index-apis/create-index/).

### file

Sink for flat file output.

Option | Required | Description
:--- | :--- | :---
path | Yes | String, path for the output file (e.g. `logs/my-transformed-log.log`).


### pipeline

Sink for writing to another pipeline.

Option | Required | Description
:--- | :--- | :---
name | Yes | String, name of the pipeline to write to.


### stdout

Sink for console output. Can be useful for testing. No options.