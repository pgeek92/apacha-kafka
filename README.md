# apacha-kafka

## Why Apache Kafka
* Created by Linkedin, now open source project mainly maintained by Confluent
* Distributed, resilient architecture, fault tolerant
* Horizontal scalability :
  * Can scale to 100s of brokers
  * Can scale to millions of messages per second
* High performance (latency of less than 10 ms) - real time
* Used by 2000+ firms, 35% of the Fortune 500

## Use cases
* Messaging system
* Activity Tracking
* Gather metrics from many different locations
* Application logs gathering
* Stream processing (with the Kafka Streams API or Spark for example)
* De-coupling of system dependencies
* Integration with Spark, Flink, Storm, Hadoop, and many other Big Data technologies

## Examples
* ***Netflix*** uses Kafka to apply recommendations in real-time while you're watching TV shows
* ***Uber*** uses Kafka to gather taxi, user and trip data in real-time to compute and forecast demand, and compute surge pricing in real-time
* ***Linkedin*** uses Kafka to prevent spam, collect user interactions to make better connection recommendations in real-time

* Remember that Kafka is only used as a transportation mechanism

## Topics, partitions and offsets
* ***Topics:*** a particular stream of data
  * Similar to a table in a database (without all the constraints
  * You can have as many topics as you want
  * A topic is identified by its name
* Topics are split in ***partitions***
  * Each partition is ordered
  * Each message within a partition gets an incremental id, called ***offset***
* Offset only have a meaning for a specific partition.
  * E.g. offset 3 in partition 0 doesn't represent the same data as offset 3 in partition 1
* Order is guaranteed only within a partition (not across partitions)
* Data is kept only for a limited time (default is one week)
* Once the data is written to a partition, ***it can't be changed*** (immutability)
* Data is assigned randomly to a partition unless a key is provided

## Brokers
* A Kafka cluster is composed of multiple brokers (servers)
* Each broker is identified with its ID (integer)
* Each broker contains certain topic partitions
* After connecting to any broker (called a bootstrap broker), you will be connected to the entire cluster
* A good number to get started is 3 brokers, but some big clusters have over 100 brokers

## Brokers and topics
* When we create a topic, kafka will automatically assign the topic and distribute it acroos all your brokers

## Topic replication factor
* Topics should have a replication factor > 1 (usually between 2 and 3)
* This way if broker is down, another broker can serve the data
* ***replication factor should be less than or equal to the total number of brokers***

### Concept of Leader for a partition
* At any time only ONE broker can be a leader for a given partition
* Only that leader can receive and serve data for a partition
* The other brokers will synchronize the data
* Therefore each partition has one leader and multiple ISR (in-sync replica)
* ***Zookeeper will decide Leader and ISRs***
* If one broker is down, in which  the partition is leader, then same partition on another broker will become the leader

## Producers
* Producers write data to topics (which is made of partitions)
* Producers automatically know to which broker and partition to write to
* In case of broker failures,Producers will automatically recover
* The load is balanced to many brokers thanks to the number of partitions
* Producers can choose to receive acknowledgement of data writes
  * ***acks=0:*** Producer won't wait for acknowledgement (possible data loss)
  * ***acks=1:*** Producer will wait for leader acknowledgement (limited data loss)
  * ***acks=all:*** Leader + replicas acknowledgement (no data loss)

### Message keys
* Producers can choose to send a ***key*** with the message (string, number, etc..)
* If key=null, data is sent round robin (broker 101 then 102 then 103...)
* If a key is sent, then all messages for that key will always go to the same partition
* A key is basically sent if you need message ordering for a specific field
* We can guarantee that a particular message having key will go this particular partition based on *key hashing*

## Consumers
* Consumers read data from a topic (identified by name)
* Consumers know which broker to read from
* In case of broker failures, consumers know how to recover
* Data is read in order ***within each partitions***

### Consumer Groups
* Consumers read data in consumer groups
* Each consumer within a group reads from exclusive partitions
* If you have more consumers tha partitions, some consumers will be inactive
* ***Consumers will automatically use a GroupCoordinator and a ConsumerCoordinator to assign a consumers to a partition***

### Consumer Groups What if too many consumers ?
* If you have more consumers than partitions, some consumers will be inactive

## Consumer Offsets
* *Kafka* stores the offsets at which consumer group has been reading
* The offsets committed live in a Kafka *topic* named __consumer_offsets__
* When a consumer in a group has processed data received from Kafka, it should be committing the offsets
* If a consumer dies, it will be able to read back from where it left off thanks to committed consumer offsets

### Delivery semantics for consumers
* Consumers choose when to commit offsets
* There are 3 delivery semantics

* *At most once:*
   * offsets are committed as soon as the message is received.
   * If the processing goes wrong, the message will be lost (it won't be read again).
* *At least once (usually preferred):*
   * offsets are committed after the message is processed.
   * If the processing goes wrong, the message will be read again.
   * This can result in duplicate processing of messages. Make sure your processing is ___Idempotent___ (i.e. processing again the messages won't impact your systems)
* *Exactly once:*
   * Can be achieved for Kafka => Kafka workflows using Kafka Streams API
   * For Kafka => External Systems workflows, use an ___Idempotent___ consumer
  
## Kafka Broker Discovery
* Every Kafka broker is also called a *bootstrap server*
* That means that ***you only need to connect to one broker*** and you will be connected to the entire cluster.
* Each broker knows about all the brokers, topics and partitions (metadata)

## Zookeeper
* Zookeeper manages brokers (keeps a list of them)
* Zookeeper helps in performing leader election for partitions
* Zookeeper sends notifications to Kafka in case of changes (e.g.new topic, broker dies, broker comes up, delete topics, etc....)
* ***Kafka can't work without zookeeper***
* Zookeeper by design operates with an odd number of servers (3, 5, 7)
* Zookeeper has a leader (handle writes) the rest of the servers are followers (handle reads)
* (Zookeeper doesn't store consumer offsets with Kafka > v0.10)

## Kafka Guarantees
* Messages are appended to a topic-partition in the order they are sent
* Consumers read messages in the order stored in a topic-partition
* With a replication factor of ***N***, producers and consumers can tolerate up to ***N-1*** brokers being down
* As long as the number of partitions remains constant for a topic (no new partitions), the same key will always go to the same partition

## Kafka Installation on windows
1. Download and Setup Java 8 JDK
2. Download the Kafka binaries from https://kafka.apache.org/downloads
3. Extract Kafka at the root of C:\
4. Setup Kafka bins in the ***Environment variables*** section by editing ***Path***
5. Try Kafka commands using `kafka-topics.bat` (for example)
6. Edit Zookeeper & Kafka configs using NotePad++ https://notepad-plus-plus.org/download/
	1. zookeeper.properties: `dataDir=C:/kafka_2.12-2.0.0/data/zookeeper` (yes the slashes are inversed)
	2. server.properties: `log.dirs=C:/kafka_2.12-2.0.0/data/kafka` (yes the slashes are inversed)
7. Start Zookeeper in one command line: `zookeeper-server-start.bat config\zookeeper.properties`
8. Start Kafka in ***another*** command line: `kafka-server-start.bat config\server.properties`

***Important: For the rest of the course, don't forget to add the extension .bat to commands being run***
*** WINDOWS Users: do not delete topics.
    * Windows has a long standing bug (KAFKA-1194) which makes Kafka crash if you delete topics, The only way to recover from this error to manually delete the folders in data/kafka`

* *Kafka Tools UI* can be used in place of Kafka CLI, which can be downloaded from 
   http://kafkatool.com/
	
