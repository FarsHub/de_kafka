[![Actions Status](https://github.com/conduktor/kafka-stack-docker-compose/workflows/CI/badge.svg)](https://github.com/conduktor/kafka-stack-docker-compose/actions)

![logo](https://raw.githubusercontent.com/conduktor/conduktor.io-public/refs/heads/main/logo/logo-signature.png)
# An open-source project by [Conduktor](https://conduktor.io/)

This project is sponsored by [Conduktor.io](https://www.conduktor.io/), the Enterprise Data Management 
Platform for Streaming. 

Once you have started your cluster, you can use Conduktor to easily manage it. 
Just connect against `localhost:9092`. If you are on Mac or Windows and want to connect from another container, use `host.docker.internal:29092`

# kafka-stack-docker-compose

This replicates as well as possible real deployment configurations, where you have your zookeeper servers and Kafka servers distinct from each other. This solves all the networking hurdles that comes with Docker and Docker Compose, and is compatible cross platform.

## Stack

  - Conduktor Platform
  - Zookeeper version
  - Kafka version
  - Kafka Schema Registry
  - Kafka Rest Proxy
  - Kafka Connect
  - ksqlDB Server
  - Zoonavigator

For a UI tool to access your local Kafka cluster, use the free version of [Conduktor](https://www.conduktor.io/get-started).

# Requirements

Kafka will be exposed on `127.0.0.1`.

## Apple M4 Support

At the time of writing there is an issue with Apple M4 chip machines and running certain Java based Docker images.

Modify the `conduktor.yml` file, uncomment the environment variable `CONSOLE_JAVA_OPTS: "-XX:UseSVE=0"`.

## Full stack

To ease you journey with Kafka just connect to [localhost:8080](http://localhost:8080/)

 - Conduktor-platform: `$DOCKER_HOST_IP:8080`
 - Single Zookeeper: `$DOCKER_HOST_IP:2181`
 - Single Kafka: `$DOCKER_HOST_IP:9092`
 - Kafka Schema Registry: `$DOCKER_HOST_IP:8081`
 - Kafka Rest Proxy: `$DOCKER_HOST_IP:8082`
 - Kafka Connect: `$DOCKER_HOST_IP:8083`
 - KSQL Server: `$DOCKER_HOST_IP:8088`
- (experimental) JMX port at `$DOCKER_HOST_IP:9001`

Run with:
```
docker compose -f full-stack.yml up
docker compose -f full-stack.yml down
```

## Single Zookeeper / Single Kafka

This configuration fits most development requirements.

 - Zookeeper will be available at `$DOCKER_HOST_IP:2181`
 - Kafka will be available at `$DOCKER_HOST_IP:9092`
 - (experimental) JMX port at `$DOCKER_HOST_IP:9999`

Run with:
```
docker compose -f zk-single-kafka-single.yml up
docker compose -f zk-single-kafka-single.yml down
```

## Single Zookeeper / Multiple Kafka

If you want to have three brokers and experiment with Kafka replication / fault-tolerance.

- Zookeeper will be available at `$DOCKER_HOST_IP:2181`
- Kafka will be available at `$DOCKER_HOST_IP:9092,$DOCKER_HOST_IP:9093,$DOCKER_HOST_IP:9094`


Run with:
```
docker compose -f zk-single-kafka-multiple.yml up
docker compose -f zk-single-kafka-multiple.yml down
```

## Multiple Zookeeper / Single Kafka

If you want to have three zookeeper nodes and experiment with zookeeper fault-tolerance.

- Zookeeper will be available at `$DOCKER_HOST_IP:2181,$DOCKER_HOST_IP:2182,$DOCKER_HOST_IP:2183`
- Kafka will be available at `$DOCKER_HOST_IP:9092`
- (experimental) JMX port at `$DOCKER_HOST_IP:9999`

Run with:
```
docker compose -f zk-multiple-kafka-single.yml up
docker compose -f zk-multiple-kafka-single.yml down
```


## Multiple Zookeeper / Multiple Kafka

If you want to have three zookeeper nodes and three Kafka brokers to experiment with production setup.

- Zookeeper will be available at `$DOCKER_HOST_IP:2181,$DOCKER_HOST_IP:2182,$DOCKER_HOST_IP:2183`
- Kafka will be available at `$DOCKER_HOST_IP:9092,$DOCKER_HOST_IP:9093,$DOCKER_HOST_IP:9094`

Run with:
```
docker compose -f zk-multiple-kafka-multiple.yml up
docker compose -f zk-multiple-kafka-multiple.yml down
```

# FAQ

## Kafka

**Q: Kafka's log is too verbose, how can I reduce it?**

A: Add the following line to your docker compose environment variables: `KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"`. Full logging control can be accessed here: https://github.com/confluentinc/cp-docker-images/blob/master/debian/kafka/include/etc/confluent/docker/log4j.properties.template

**Q: How do I delete data to start fresh?**

A: Your data is persisted from within the docker compose folder, so if you want for example to reset the data in the full-stack docker compose, do a `docker compose -f full-stack.yml down`.

**Q: Can I change the zookeeper ports?**

A: yes. Say you want to change `zoo1` port to `12181` (only relevant lines are shown):
```
  zoo1:
    ports:
      - "12181:12181"
    environment:
        ZOO_PORT: 12181
        
  kafka1:
    environment:
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:12181"
```

**Q: Can I change the Kafka ports?**

A: yes. Say you want to change `kafka1` port to `12345` (only relevant lines are shown). Note only `LISTENER_DOCKER_EXTERNAL` changes:
```
  kafka1:
    image: confluentinc/cp-kafka:7.2.1
    hostname: kafka1
    ports:
      - "12345:12345"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka1:19092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:12345,DOCKER://host.docker.internal:29092
```

**Q: Kafka is using a lot of disk space for testing. Can I reduce it?**

A: yes. This is for testing only!!! Reduce the KAFKA_LOG_SEGMENT_BYTES to 16MB and the KAFKA_LOG_RETENTION_BYTES to 128MB

```
  kafka1:
    image: confluentinc/cp-kafka:7.2.1
    ...
    environment:
      ...
      # For testing small segments 16MB and retention of 128MB
      KAFKA_LOG_SEGMENT_BYTES: 16777216
      KAFKA_LOG_RETENTION_BYTES: 134217728
```

**Q: How do I expose Kafka?**

A: If you want to expose Kafka outside of your local machine, you must set `KAFKA_ADVERTISED_LISTENERS` to the IP of the machine so that Kafka is externally accessible. To achieve this you can set `LISTENER_DOCKER_EXTERNAL` to the IP of the machine.
For example, if the IP of your machine is `50.10.2.3`, follow the sample mapping below:

```
  kafka1:
    image: confluentinc/cp-kafka:7.2.1
    ...
    environment:
      ...
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka2:19093,EXTERNAL://50.10.2.3:9093,DOCKER://host.docker.internal:29093
```

**Q: How do I add connectors to Kafka connect?**

Create a `connectors` directory and place your connectors there (usually in a subdirectory) `connectors/example/my.jar`

The directory is automatically mounted by the `kafka-connect` Docker container

OR edit the bash command which pulls connectors at runtime

```
confluent-hub install --no-prompt debezium/debezium-connector-mysql:latest
        confluent-hub install 
```

**Q: How to disable Confluent metrics?**

Add this environment variable
```
KAFKA_CONFLUENT_SUPPORT_METRICS_ENABLE=false
```
