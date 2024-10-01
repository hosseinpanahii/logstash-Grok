# opensearch-logstash

## Logstash Service Architecture
Logstash processes logs from different servers and data sources and it behaves as the shipper. The shippers are used to collect the logs and these are installed in every input source. Brokers like Redis, Kafka or RabbitMQ are buffers to hold the data for indexers, there may be more than one brokers as failed over instances.

Indexers like Lucene are used to index the logs for better search performance and then the output is stored in Elasticsearch or other output destination. The data in output storage is available for Kibana and other visualization software.

![alt text](https://github.com/hosseinpanahii/opensearch-logstash/blob/main/logstash_service_architecture.png)
