# Docker packages(Elastic stack) for easy monitoring of kafka brokers at one-click 
- Docker-compose로 제공되는 elastic stack을 활용하여, 다수의 kafka broker에서 제공하는 jmx metric을 빠르게 수집할 환경 구성
- 전체 과정은 docker-compose up 실행 한번으로 구동 가능 
- broker의 ip(hostname)만 설정하면, 자동으로 기본 jmx metric을 수집하여 elasticsearch에 저장
  - 수집할 jmx metric은 업무별로 추가/삭제 가능
- 사용자는 kibana에 접속하여 기본 dashboard를 이용하여 모니터링 가능 (각 업무별로 시각화 추가/변경 가능)
- elastic stack을 구성하는 기본 docker-compose 설정은 docker-elk(https://github.com/deviantony/docker-elk)를 활용함. 

## STEP 1. 시스템 환경 설정 
- docker-compose가 사전에 설치되어 있어야 함. 
- Host(Server) 최소 구성요건
```
Docker Engine version 17.05 or newer
Docker Compose version 1.20.0 or newer
RAM : 1.5 GB of RAM
```
- 본 docker-compose package를 설치할 서버로 접속하여, 아래 절차를 수행한다. 

### Install doker engine 
- https://docs.docker.com/engine/install/centos/
```
> sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

> sudo yum install -y yum-utils

> sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

> sudo yum install -y docker-ce docker-ce-cli containerd.io
> sudo systemctl start docker
> sudo systemctl enable docker

# 설치된 docker version 확인 
> sudo docker version
Client: Docker Engine - Community
 Version:           20.10.12

Server: Docker Engine - Community
 Engine:
  Version:          20.10.12

# docker 실행시 사용자 권한으로 실행하기 위한 권한 변경
# 아래 명령어 실행 후 콘솔을 재시작 해야 함. (로그아웃 후 다지 접속)
> sudo usermod -aG docker user_id
```

### Install docker-compose
- https://docs.docker.com/compose/install/
```
> cd ~
> sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
> sudo chmod +x /usr/local/bin/docker-compose
> docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

## STEP 2. kafka broker의 jmx metric 수집하기
### Broker가 실행중인 서버의 IP/Hostname으로 변경 
```
> cd ~
> git clone https://github.com/freepsw/kafka-metrics-monitoring
> cd ~/kafka_monitoring_package/docker-elk

# Broker 접속을 위한 ip/hotname 변경 
> vi logstash/pipeline/jmx_conf/broker01.conf
> > vi logstash/pipeline/jmx_conf/broker01_resources.conf
## 아래 정보 중 host 정보를 본인이 운영중인 broker 서버의 ip 또는 hostname으로 변경한다. 
  "host" : "broker-01",
  "port" : 9999,
  ...
```
- broker가 여러 서버로 구성되어 있고, 이를 다 모니터링하려면, 
- broker01.conf 파일을 복사하여, broker02.conf로 새로운 파일을 생성하고, hostname/ip를 변경해 준다. 

### JMX metric 수집하기
- Docker image(elasticsearch, kibana, logstash)를 build하고 시간이 소요됨. (NW 성능에 따라 5분 이상 소요)
- docker image가 build되면, 각 프로그램들을 실행하며, logstash에서 kafka broker에 접속하여 jmx metric을 수집한다. 
```
> cd ~/kafka_monitoring_package/docker-elk
> docker-compose up

### 아래와 같은 메세지가 출력되면 정상적으로 데이터를 수집하는 것임. 
logstash_1       | {
logstash_1       |                "@version" => "1",
logstash_1       |                    "host" => "broker-01",
logstash_1       |              "@timestamp" => 2022-01-09T01:38:35.710Z,
logstash_1       |                    "path" => "/usr/share/logstash/pipeline/jmx_conf",
logstash_1       |                    "type" => nil,
logstash_1       |             "metric_path" => "broker01.ReplicaManager.UnderReplicatedPartitions.Value",
logstash_1       |     "metric_value_number" => 0
logstash_1       | }
```

## STEP 3. Kibana 시각화를 위한 설정 
- Kibana web ui에서 실행할 수도 있으나, console에서 빠르게 필요한 명령어 실행 가능
### Kibana index pattern(kafka_mon) 생성
- 위의 docker-compose up을 실행하면, 
- elasticsearch에 "kafka_mon"이라는 index를 생성하고, 수집된 jmx metric을 저장한다. 
- 아래 명령어는 kibana web ui에서도 사용자가 할 수 있으나, console에서 빠르게 처리할 수 있는 명령어이다. 
```
> curl --user elastic:changeme -X POST "localhost:5601/api/index_patterns/index_pattern" -H 'kbn-xsrf: true' -H 'Content-Type: application/json' -d'
{
  "index_pattern": {
     "title": "kafka_mon",
     "timeFieldName": "@timestamp"
  }
}'
```

### Import kibana dashboard for kafka (사전에 생성된 kafka broker 모니터링용 dashboard)
- 간략하게 확인 가능한 kafka monitoring 용 dashboard
```
curl --user elastic:changeme -X POST "34.64.209.155:5601/api/saved_objects/_import" -H "kbn-xsrf: true" --form file=@kibana_dashboard.ndjson

file.ndjson
{"type":"index-pattern","id":"my-pattern","attributes":{"title":"my-pattern-*"}}
{"type":"dashboard","id":"my-dashboard","attributes":{"title":"Look at my dashboard"}

```

### Kibana Web UI 접속 (dashboard 확인)
- Browser에서 <Mointoring 서버 IP>:5601로 접속
  - id : elastic
  - password : changeme 


## STEP 5. Stop docker-compose
- 기존에 실행되었던 container image의 volume 데이터까지 삭제하려면, -v 옵션을 추가
```
> docker-compose down

# 기존에 실행되었던 container image의 volume 데이터까지 삭제하려면, -v 옵션을 추가
# 이 경우, elasticsearch에 수집된 metric 데이터와 kibana의 그래프(차트, 대시보드)도 삭제된다.
> docker-compose down -v
```

## 현재 프로젝트에 대한 추가 설명 
### 1. 기반이 되는 github 프로젝트
- 본 코드는 docker-elk라른 github 코드에서 jmx 설정 및 라이센스 관련 부분을 수정하여 구성됨
- docker-elk는 github에 공개된 elastic stack 설치 스크립트
  - 최신 elastic stack을 설치해 주며, basic 또는 trial(상용 30일 무료) license를 적용하여 실행 가능
  - 본 스크립트에서는 기본으로 basic으로 실행 (elasticsearch/config/elasticsearch.yml)
  - https://github.com/deviantony/docker-elk 참고 

#### 주요 변경 코드
- elasticsearch license 변경 : elasticsearch/config/elasticsearch.yml
  - trial --> basic
- logstash plugin 설치 : logstash/Dockerfile
  - RUN logstash-plugin install logstash-input-jmx
- logstash broker jmx 설정 추가
  - logstash/pipeline/jmx_conf : 수집할 jmx metric 
  - logstash/pipeline/logstash_jmx.conf : jmx metric을 수집하여 elasticsearch에 저장하는 pipeline 설정

## ETC (기타 설정)

### Elastic stack version 변경은?
- 원하는 elastic stack 버전으로 실행하려면, 
- github의 branch(https://github.com/deviantony/docker-elk/branches)에서 원하는 버전을 선택하여, git clone을 한다. 
  - 현재 권장하는 버전은 7.x, 6.x 버전이다. 
- 다른 방식으로는 
  - 1) .env에서 ELK_VERSION=7.16.2로 지정된 버전을 직접 변경하는 방법
  - 2) 각 제품의 Dockerfile에서 직접 버전을 지정하는 방법
       - 이 방식은 제품간의 버전 차이가 발생하면, 호환성 문제 등 사용자가 직접 모든 부분을 해결해야 한다. 
```
> cd ~/
```


### 기본 패스워드 변경 
-  https://github.com/deviantony/docker-elk#how-to-reset-a-password-programmatically 참고
```
>  curl -XPOST -D- 'http://localhost:9200/_security/user/elastic/_password' \
    -H 'Content-Type: application/json' \
    -u elastic:<your current elastic password> \
    -d '{"password" : "<your new password>"}'
```

### Java JVM 설정 변경
- https://github.com/deviantony/docker-elk#jvm-tuning
- docker-compose.yml 설정 파일에서 수정 가능
```yml
logstash:

  environment:
    LS_JAVA_OPTS: -Xmx1g -Xms1g
```

### 강제로 모든 docker image 삭제하기
- 현재 서버의 모든 docker image가 삭제되므로, 주의해서 사용
```
> docker rmi -f $(docker images -q)
```

-daemon
-daemon