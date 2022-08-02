# Test loggie for Kafka source via golang

## Required
- Docker
- Docker compose
- [loggie](https://github.com/loggie-io/loggie) 

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
make build # you can get the executable file after about 5 mins
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
> ./loggie -config.system=./loggie.yml -config.pipeline=./pipelines.yml
```
