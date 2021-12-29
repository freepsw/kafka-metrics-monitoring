# STEP1. Elasticsearch 설치 
- "kafka-monitoring" server로 접속하여 ch.04 > 01 에서 가이드된 명령어 실행

# STEP 2. Install the kibana
- "kafka-monitoring" server로 접속하여 ch.04 > 01 에서 가이드된 명령어 실행


# STEP 3. Install the metricbeat
## Install the logstash and run
```
> cd ~
> curl -OL https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-oss-7.10.2-linux-x86_64.tar.gz
> tar xvf metricbeat-oss-7.10.2-linux-x86_64.tar.gz
> rm -rf metricbeat-oss-7.10.2-linux-x86_64.tar.gz
> cd ~/metricbeat-7.10.2-linux-x86_64
```

## Set up kafka module 
- metricbeat은 각 모듈별로 다양한 지원(시각화, 설정 등)을 한다. 
- 이를 위해서는 각 모듈을 사용할 수 있도록 아래 설정이 필요
```
## 현재 활성돠된(사용 가능한) module을 확인한다. 
## kafka는 기본으로 비활성화되어 있음. 
> cd ~/metricbeat-7.10.2-linux-x86_64
> ./metricbeat modules list
Enabled:
system

Disabled:
apache
docker
http
kafka
....

## kafak module 활성화
> ./metricbeat modules enable kafka
Enabled kafka

## kafka module이 활성화 되었는지 확인 
> ./metricbeat modules list
Enabled:
kafka
system


## setup 명령어를 실행하면, metricbeat에서 수집한 데이터를 시각화할 화면과 index를 자동을 로딩한다. 
> ./metricbeat setup -e kafka
........
Index setup finished.
Loading dashboards (Kibana must be running and reachable)
2021-12-09T06:57:46.315Z	INFO	kibana/client.go:119	Kibana url: http://localhost:5601
2021-12-09T06:57:46.658Z	INFO	kibana/client.go:119	Kibana url: http://localhost:5601

2021-12-09T06:58:26.564Z	INFO	instance/beat.go:815	Kibana dashboards successfully loaded.
Loaded dashboards
```

# STEP 4. Install the metricbeat
## kafka module에서 사용할 config를 수정
- metric을 수집할 broker의 ip:port를 수정 
```
## metric
> cd ~/metricbeat-7.10.2-linux-x86_64
> vi modules.d/kafka.yml
- module: kafka
  #metricsets:
  #  - partition
  #  - consumergroup
  period: 3s
  hosts: ["broker-01:9092"]
```

## Run the metircbeat
```
> cd ~/metricbeat-7.10.2-linux-x86_64
> ./metricbeat -e
```

## 생성된 index (metricbeat-7.10.2-2021.12.10) 확인 
- 자동으로 kibana에서 시각화할 metric을 저장할 index가 생성된 것을 확인
```
> curl -X GET "localhost:9200/_cat/indices/"
yellow open metricbeat-7.10.2-2021.12.10 7L6dT3IMQu6TgdMYSBufvg 1 1   0  0    208b    208b
```


# STEP 5. GCP 방화벽 허용 PORT 추가 
- Kibana 접속하기 위한 5601 PORT 허용

# STEP 4. Web Browser로 접속
- http://<kafka-monitoring vm IP>:5601






