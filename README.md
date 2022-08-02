# Test loggie for Kafka source via golang

## Required
- Docker
- Docker compose
- [loggie](https://github.com/loggie-io/loggie) 

## Local Kafka Preparation

Use Docker Compose(make sure you have installed the version that can support version 3) to create a one-zk-one-kafka local kafka
1. create the docker-compose file
```yaml
---
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.0.0
    hostname: zookeeper
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: confluentinc/cp-kafka:7.0.0
    container_name: broker
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,PLAINTEXT_INTERNAL://broker:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1

```

2. `docker-compose up` for kafka server starts
```shell
> docker-compose up -d .

```


## Loggie Preparation

Use loggie binary for test:

1. download loggie code
```shell
git clone https://github.com/loggie/loggie.git
```
2. compile into the binary
```shell
export GOOS="linux"
export GOARCH="amd64"
make build # you can get the executable file "loggie" after about 5 mins
```
3. create two basic configuration files
```yaml
### ==> loggie.yaml
loggie:
  monitor:
    logger:
      period: 30s
      enabled: true
    listeners:
      filesource: ~
      filewatcher: ~
      reload: ~
      sink: ~

  reload:
    enabled: true
    period: 10s

  http:
    enabled: true
    port: 9196

### ==> pipelines.yaml 
pipelines:
  - name: local
    sources:
      - type: kafka
        name: local-kafka
        brokers: ["127.0.0.1:9092"] # use the local kafka
        topic: purchases # use the specifed topic
    sink:
      type: dev
      printEvents: true
      codec:
        pretty: true
```
4. start the service
```shell
> ./loggie -config.system=./loggie.yaml -config.pipeline=./pipelines.yaml

2022-08-02 06:54:34 INF cmd/loggie/main.go:64 > version: main-ce454a8
2022-08-02 06:54:34 INF cmd/loggie/main.go:73 > real GOMAXPROCS 2
2022-08-02 06:54:34 INF cmd/loggie/main.go:76 > node name: api-ng
2022-08-02 06:54:34 INF pkg/eventbus/center.go:124 > listener(filesource) start
2022-08-02 06:54:34 INF pkg/eventbus/center.go:124 > listener(filewatcher) start
2022-08-02 06:54:34 INF pkg/eventbus/center.go:124 > listener(reload) start
2022-08-02 06:54:34 INF pkg/eventbus/center.go:124 > listener(sink) start
2022-08-02 06:54:34 INF cmd/loggie/main.go:89 > pipelines config path: ./pipelines.yml
2022-08-02 06:54:34 INF cmd/loggie/main.go:99 > initial pipelines:
pipelines:
- name: local
  cleanDataTimeout: 5s
  queue:
    name: default
    type: channel
    batchSize: 2048
  interceptors:
  - type: metric
  - type: maxbytes
  - type: retry
  sources:
  - name: local-kafka
    type: kafka
    brokers:
    - 127.0.0.1:9092
    topic: purchases
    fieldsUnderKey: fields
  sink:
    type: dev
    printEvents: true
    parallelism: 1
    codec:
      type: json
      pretty: true
2022-08-02 06:54:34 INF pkg/control/controller.go:61 > starting pipeline: local
2022-08-02 06:54:34 INF pkg/sink/dev/sink.go:77 > sink/dev start
...
```
