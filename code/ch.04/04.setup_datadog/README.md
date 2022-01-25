# STEP1. Run the producer & consumer  
- "kafka-client" 서버에 접속하여 아래 명령어 실행
```
## run the producer 
> cd ~
> export LS_JAVA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.port=9998 
  -Dcom.sun.management.jmxremote.rmi.port=9998
  -Djava.rmi.server.hostname=34.64.168.235'
> ~/logstash-7.15.0/bin/logstash -w 1 -b 1 -f ~/logstash_conf/producer.conf

## run the consumer
> export LS_JAVA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.port=9997 
  -Dcom.sun.management.jmxremote.rmi.port=9997
  -Djava.rmi.server.hostname=34.64.168.235'
> ~/logstash-7.15.0/bin/logstash -w 1 -b 1 --path.data ~/data/consumer_data -f ~/logstash_conf/consumer.conf
```
  <!-- -Djava.rmi.server.hostname=34.64.185.183' -->

# STEP 2. Install the datadog agent (centos)
- "broker-01" 서버에 접속하여 아래 명령어 실행
```
## datadog agent 설치 (centos 용)
> DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=90fb7058e7xxxxxxxxxx DD_SITE="datadoghq.com" bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"
..........
If you ever want to stop the Agent, run:
    sudo systemctl stop datadog-agent
And to run it again run:
    sudo systemctl start datadog-agent

## datadog agent가 정상 동작 중인지 확인
> ps -ef | grep datadog
dd-agent 10384     1  1 09:13 ?        00:00:03 /opt/datadog-agent/bin/agent/agent run -p /opt/datadog-agent/run/agent.pid
dd-agent 10385     1  0 09:13 ?        00:00:00 /opt/datadog-agent/embedded/bin/trace-agent --config /etc/datadog-agent/datadog.yaml --pid /opt/datadog-agent/run/trace-agent.pid
dd-agent 10386     1  0 09:13 ?        00:00:00 /opt/datadog-agent/embedded/bin/process-agent --config=/etc/datadog-agent/datadog.yaml --sysprobe-config=/etc/datadog-agent/system-probe.yaml --pid=/opt/datadog-agent/run/process-agent.pid

> sudo systemctl status datadog-agent

```

# STEP 2. Apache Kafka 모니터링에 필요한 datadog agent 설정 변경

## Kafka Broker/Producer/Consumer JMX 설정 추가
- JMX로 모니터링할 서버와 jmx port를 지정한다. 
```
> sudo vi /etc/datadog-agent/conf.d/kafka.d/conf.yaml
# 아래 내용 추가 
init_config:

    is_jmx: true
    collect_default_metrics: true
    new_gc_metrics: true

instances:

  - host: localhost
    port: 9999
    tags:
      - kafka: broker0

  - host: kafka-client
    port: 9997
    tags:
      - kafka: consumer0

  - host: kafka-client
    port: 9998
    tags:
      - kafka: producer0
```
## Kafka Consumer 관련(lag, offet 등) JMX 설정 추가
- Consumer 관련 내용만 수집
- https://docs.datadoghq.com/integrations/kafka/?tab=host#kafka-consumer-integration 참고
```
> sudo vi /etc/datadog-agent/conf.d/kafka_consumer.d/conf.yaml 
# 아래 내용 추가
init_config:
    # kafka_timeout: 5

instances:

  - kafka_connect_str:
    - localhost:9092
    kafka_consumer_offsets: true
    monitor_unlisted_consumer_groups: true
```

# STEP 4. Datadog agent 재시작
```
## 서비스 재시작
> sudo systemctl restart datadog-agent

## 서비스 상태 확인
> sudo datadog-agent status
# 아래 kafka의 정보가 정상적으로 보이면 됨.
=========
Collector
=========

  Running Checks
  ==============
    kafka_consumer (2.12.1)
    -----------------------
      Instance ID: kafka_consumer:23a55777eda24f89 [OK]
      Configuration Source: file:/etc/datadog-agent/conf.d/kafka_consumer.d/conf.yaml
      Total Runs: 1
      Metric Samples: Last Run: 6, Total: 6
      Events: Last Run: 0, Total: 0
      Service Checks: Last Run: 0, Total: 0
      Average Execution Time : 617ms
      Last Execution Date : 2021-12-09 12:29:07 UTC (1639052947000)
      Last Successful Execution Date : 2021-12-09 12:29:07 UTC (1639052947000)

========
JMXFetch
========

  Information
  ==================
    runtime_version : 1.8.0_312
    version : 0.44.3
  Initialized checks
  Information
  ==================
    runtime_version : 1.8.0_312
    version : 0.44.3
  Initialized checks
  ==================
    kafka
      instance_name : kafka-localhost-9999
      message : <no value>
      metric_count : 69
      service_check_count : 0
      status : OK
      instance_name : kafka-kafka-client-9997
      message : <no value>
      metric_count : 34
      service_check_count : 0
      status : OK
      instance_name : kafka-kafka-client-9998
      message : <no value>
      metric_count : 56
      service_check_count : 0
      status : OK
```



# references
- https://www.datadoghq.com/blog/monitor-kafka-with-datadog/
- https://docs.datadoghq.com/integrations/faq/troubleshooting-and-deep-dive-for-kafka/