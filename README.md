# opensearch-logstash

## Logstash Service Architecture
Logstash processes logs from different servers and data sources and it behaves as the shipper. The shippers are used to collect the logs and these are installed in every input source. Brokers like Redis, Kafka or RabbitMQ are buffers to hold the data for indexers, there may be more than one brokers as failed over instances.

Indexers like Lucene are used to index the logs for better search performance and then the output is stored in Elasticsearch or other output destination. The data in output storage is available for Kibana and other visualization software.

![alt text](https://github.com/hosseinpanahii/opensearch-logstash/blob/main/logstash_service_architecture.png)

## Logstash Internal Architecture

The Logstash pipeline consists of three components Input, Filters and Output. The input part is responsible to specify and access the input data source such as the log folder of the Apache Tomcat Server.
![alt text](https://github.com/hosseinpanahii/opensearch-logstash/blob/main/logstash_internal_architecture.png)

## Example to Explain the Logstash Pipeline
The Logstash configuration file contains the details about the three components of Logstash. In this case, we are creating a file name called Logstash.conf.

The following configuration captures data from an input log “inlog.log” and writes it to an output log “outlog.log” without any filters.

## Logstash.conf
The Logstash configuration file just copies the data from the inlog.log file using the input plugin and flushes the log data to outlog.log file using the output plugin.
---bash
input {
   file {
      path => "C:/tpwork/logstash/bin/log/inlog.log"
   }
}
output {
   file {
      path => "C:/tpwork/logstash/bin/log/outlog.log"
   }
}
---
