# STEP1. Run the producer & consumer  
- "kafka-client" 서버에 접속하여 아래 명령어 실행
```
## run the producer 
> cd
> export LS_JAVA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.port=9998 
  -Dcom.sun.management.jmxremote.rmi.port=9998'
> ~/logstash-7.15.0/bin/logstash -w 1 -b 1 -f ~/logstash_conf/producer.conf

## run the consumer
> export LS_JAVA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.port=9997 
  -Dcom.sun.management.jmxremote.rmi.port=9997'
> ~/logstash-7.15.0/bin/logstash -w 1  --path.data ~/data/consumer_data -f ~/logstash_conf/consumer.conf
```

# STEP 5. Run the logstash to collect jmx metrics
```
> cd ~/logstash-7.10.2/
> bin/logstash -f ~/logstash_conf/logstash_jmx.conf
```