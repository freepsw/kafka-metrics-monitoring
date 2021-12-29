# STEP 0. Connect to vm server using ssh
- "kafka-client" 서버로 접속한 후, 아래의 명령어를 순서대로 실행한다. 


# STEP1. Run the logstash as a kafka consumer.
- Producer와 Consumer를 동일한 서버에 설치하는 경우,
- 아래의 java 설치, logstash 설치 과정은 생략 가능.
## Java 설치 및 JAVA_HOME 설정
## Install the logstash(Kafka consumer) and run logstash with the jmx option enabled
### Install the logstash and run
### Configure the logstash.yml


### Run the logstash(consumer) 
```
## Add consumer config file 
> mkdir ~/logstash_conf
> vi ~/logstash_conf/consumer.conf

## Run logstash as a kafka consumer
> export LS_JAVA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.port=9997 
  -Dcom.sun.management.jmxremote.rmi.port=9997 
  -Djava.rmi.server.hostname=34.64.185.183'

## thread 1개, batch size 1개로 데이터를 kafka로 전송
## logstash 실행시 -b (batch.size) 옵션을 1로 설정해야, 
## broker에 데이터가 1건이라로 도착하면, logstash에서 consumer thread로 데이터를 전송하고, 이를 화면에 출력한다. 
## -b 128로 하면, broker에서 128건을 가져올 때 까지 기다린 후 화면에 출력.
> mkdir ~/data
> ~/logstash-7.15.0/bin/logstash -w 1 -b 1 --path.data ~/data/consumer_data -f ~/logstash_conf/consumer.conf
```

#### LS_JAVA_OPT 설정을 logstash 시작시에 기본으로 적용하는 방식
```
> vi logstash-7.15.0/config/jvm.options

## 위 파일 내용에 아래 jmx 관련된 내용을 LS_JAVA_OPTS에 추가
-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=9998 -Dcom.sun.management.jmxremote.rmi.port=9998 -Djava.rmi.server.hostname=34.64.239.87
```


# STEP3. Check the consumer metrics using the JConsole
- JDK가 설치된 노트북에 아래 명령어 실행. 
```
> jconsole
```