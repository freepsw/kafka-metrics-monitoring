# STEP 0. Connect to vm server using ssh
- "kafka-client" 서버로 접속한 후, 아래의 명령어를 순서대로 실행한다. 

# STEP1. Run the logstash as a kafka producer.

## Java 설치 및 JAVA_HOME 설정
```
> sudo yum install -y java

# 현재 OS 설정이 한글인지 영어인지 확인한다. 
> alternatives --display java

# 아래와 같이 출력되면 한글임. 
슬레이브 unpack200.1.gz: /usr/share/man/man1/unpack200-java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64.1.gz
현재 '최고' 버전은 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre/bin/java입니다.

### 한글인 경우 
> alternatives --display java | grep '현재 /'| sed "s/현재 //" | sed 's|/bin/java로 링크되어 있습니다||'
> export JAVA_HOME=$(alternatives --display java | grep '현재 /'| sed "s/현재 //" | sed 's|/bin/java로 링크되어 있습니다||')

### 영문인 경우
> alternatives --display java | grep current | sed 's/link currently points to //' | sed 's|/bin/java||' | sed 's/^ //g'
> export JAVA_HOME=$(alternatives --display java | grep current | sed 's/link currently points to //' | sed 's|/bin/java||' | sed 's/^ //g')

# 제대로 java 경로가 설정되었는지 확인
# 아래 명령어를 실행하면, "/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre"와 유사한 경로가 보여야 함. 
> echo $JAVA_HOME

> echo "export JAVA_HOME=$JAVA_HOME" >> ~/.bash_profile
> source ~/.bash_profile
```


## Install the logstash(Kafka producer) and run logstash with the jmx option enabled
### Download the test file (tracks.csv)
- Procuder에서 broker로 전송할 데이터를 다운로드 받는다. 
```
> curl -OL https://raw.githubusercontent.com/freepsw/demo-spark-analytics/master/00.stage1/tracks.csv
> mv tracks.csv /tmp/tracks.csv
```

### Install the logstash and run
```
> cd ~
> curl -LO https://artifacts.elastic.co/downloads/logstash/logstash-oss-7.15.0-linux-x86_64.tar.gz
> tar xvf logstash-oss-7.15.0-linux-x86_64.tar.gz  
> rm -rf logstash-oss-7.15.0-linux-x86_64.tar.gz
> cd logstash-7.15.0
```

### Configure the logstash.yml
- kafka producer로 동작하기 위해서, 일정 주기로 일정 메세지를 전달하도록 설정 필요.
- logstash를 기본으로 사용하면, 전송 성능을 위해서 정의된 batch size 만큼을 보내도록 설정됨. 
- 따라서 본 실습을 위해서는 1개의 thread가 한번에 1개의 메세지만 보내도록 설정한다. 
- 관련 configuration
```yml
# logstash에서 동시에 실행 가능한 thread
pipeline.workers: 2
# logstash에서 한번에 전송할 batch size
pipeline.batch.size: 125
```

- logstash 실행시 적용 가능한 option 
```
thread 게수 : -w 또는 --pipeline.workers
batch 개수 : -b, --pipeline.batch.siz
```


### Run the logstash 
```
## Add producer config file 
> mkdir ~/logstash_conf
> vi ~/logstash_conf/producer.conf

## Run logstash as a kafka producer
> export LS_JAVA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.port=9998 
  -Dcom.sun.management.jmxremote.rmi.port=9998 
  -Djava.rmi.server.hostname=34.64.185.183'

## thread 1개, batch size 1개로 데이터를 kafka로 전송
> ~/logstash-7.15.0/bin/logstash -w 1 -b 1 -f ~/logstash_conf/producer.conf
```

#### LS_JAVA_OPT 설정을 logstash 시작시에 기본으로 적용하는 방식
```
> vi logstash-7.15.0/config/jvm.options

## 위 파일 내용에 아래 jmx 관련된 내용을 LS_JAVA_OPTS에 추가
-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=9998 -Dcom.sun.management.jmxremote.rmi.port=9998 -Djava.rmi.server.hostname=34.64.239.87
```


# STEP3. Check the producer metrics using the JConsole
- JDK가 설치된 노트북에 아래 명령어 실행. 
```
> jconsole
```
