Mozilla Nginx Data Ingestion
----------------------------

## Overview
Nginx module for taking HTTP requests, transforming them into Heka
protobuf messages and sending them directly to a Kafka cluster.

### Prerequisites
* [forked librdkafka](https://github.com/trink/librdkafka/tree/dr_no_poll)
* [Lua Sandbox](https://github.com/mozilla-services/lua_sandbox)

### Build Instructions
    
    # from the nginx build directory
    ./configure --add-module=<repo_path>/nginx_moz_ingest --prefix=<nginx_path>/nginx-1.7.0

### Example Configuration
```
worker_processes  4;
daemon off;
error_log  logs/error.log;

events {
    worker_connections  1024;
}


http {
    default_type  application/octet-stream;
    sendfile        on;
    server {
        listen       8880;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }

        moz_ingest_kafka_brokerlist         localhost:9092;
        moz_ingest_kafka_max_buffer_size    100000;
        moz_ingest_kafka_max_buffer_ms      10;
        moz_ingest_kafka_batch_size         100;
        moz_ingest_client_ip                on;
        moz_ingest_max_unparsed_uri_size    32;
        moz_ingest_max_content_size         100k;
        moz_ingest_header                   Content-Length;
        moz_ingest_header                   User-Agent;

        location /telemetry/ {
                   keepalive_timeout 0;
                   moz_ingest;
                   moz_ingest_kafka_topic test;
        }
    }
}
```

### Directives

#### moz_ingest
Activates the handler for the specified location.

    Syntax: moz_ingest;
    Default: --
    Context: location

#### moz_ingest_kafka_brokerlist
Specifies the Kafka broker list as specified by the 
[librdkafka documentation](https://github.com/trink/librdkafka/blob/dr_no_poll/src/rdkafka.h#L2258)

    Syntax: moz_ingest_kafka_brokerlist string;
    Default: --
    Context: main, http, server, location

#### moz_ingest_kafka_brokerlist
Specifies the Kafka topic name to send the messages to.

        Syntax: moz_ingest_kafka_topic string;
        Default: --
        Context: main, http, server, location

#### moz_ingest_kafka_max_buffer_size
Specifies the maximum number of messages allowed on the Kafka producer queue.

    Syntax: moz_ingest_kafka_max_buffer_size size;
    Default: 100000
    Context: main, http, server, location

#### moz_ingest_kafka_max_buffer_ms
Specifies the maximum time, in milliseconds, for buffering data on the Kafka producer queue.

    Syntax: moz_ingest_kafka_max_buffer_ms time;
    Default: 1000
    Context: main, http, server, location

#### moz_ingest_kafka_batch_size
Specifies the maximum number of messages batched in one Kafka MessageSet.

    Syntax: moz_ingest_kafka_batch_size time;
    Default: 1000
    Context: main, http, server, location

#### moz_ingest_client_ip
Specifies whether or not the client IP address should be included as a **remote_addr** message field.

    Syntax: moz_ingest_client_ip on | off;
    Default: on
    Context: main, http, server, location

#### moz_ingest_max_unparsed_uri_size
Specifies the maximum unparsed URI size to accept before returning [414].

    Syntax: moz_ingest_max_unparsed_uri_size size;
    Default: 256
    Context: main, http, server, location

#### moz_ingest_max_content_size
Specifies the maximum content size to accept before returning [413].

    Syntax: moz_ingest_max_content_size size;
    Default: 100k
    Context: main, http, server, location

#### moz_ingest_header
Specifies a header that should be included as message field (if it exists).

    Syntax: moz_ingest_header field;
    Default: --
    Context: main, http, server, location
