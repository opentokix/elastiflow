### 6b. Configure Kafka (Advanced) [NOT REQUIRED]

This system depends on you having a kafka system running to begin with, and that you are comfortable running.

Kafka can be used as a buffering, and load balancing for high volume flows of data. This would be considered an advanced configuration where you divide the elastiflow system onto two system. 

You have to make some changes to the configuration files in order to get SSL transport, and manually create topics in kafka and such. 

**Settings to take note of:**

acks: How many kafka servers should ack the message before it is "delivered" 
backoff_ms: Number of milliseconds to wait to reconnect. 

**Original Design:**
[Flow source] -> [LogStash filtering] -> [ES]

**Kafka Design:**
[Flow source] -> [LogStash] --(json)--> [Kafka] --(json)--> [LogStash filtering] -> [ES]

* [Kafka output](logstash/elastiflow/conf.d/15_kafka_tansport.logstash.conf.disabled)
* [Kafka input](logstash/elastiflow/conf.d/18_kafka_transport.logstash.conf.disabled)

Replication factor can not be larger than your available number of brokers.

If you have nine partitions, you will be able to have nine parallell logstash instances working on events in kafka. To increase the thruput. You can also horizontally scale both kafka cluster and the logstash ingests. 

```
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 9 --topic ipfix
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 9 --topic sflow
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 9 --topic netflow

# Setting retention of topics to 24 hours (This can be A LOT of data)
bin/kafka-configs.sh --zookeeper kafka1:2181 --alter --entity-type topics --topic ipfix --add-config retention.ms=86400000
bin/kafka-configs.sh --zookeeper kafka1:2181 --alter --entity-type topics --topic sflow --add-config retention.ms=86400000
bin/kafka-configs.sh --zookeeper kafka1:2181 --alter --entity-type topics --topic netflow --add-config retention.ms=86400000

```

**Logstash workers**

This configuration will let you use up to nine workers to filter and consume data. This can be running in kubernetes or some other container orchestration system. 