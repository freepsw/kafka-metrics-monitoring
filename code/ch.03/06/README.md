# STEP 0. Connect to kafka-monitoring server using ssh
- "kafka-monitoring" 서버로 접속한 후, 아래의 명령어를 순서대로 실행한다. 

# STEP1. CMAK 설치

## Install the java 11 
- CMAK 3.0.05는 java 11 부터 설치가 가능함. 
```
## 설치 가능한 java version 확인
> yum list java*jdk-devel
Available Packages
java-1.6.0-openjdk-devel.x86_64
java-1.7.0-openjdk-devel.x86_64 
java-1.8.0-openjdk-devel.i686 
java-1.8.0-openjdk-devel.x86_64  
java-11-openjdk-devel.i686  
java-11-openjdk-devel.x86_64
java-latest-openjdk-devel.x86_64 

## jdk 11 설치
> sudo yum install -y java-11-openjdk-devel.x86_64

## 설치된 java version 확인
> java -version
openjdk version "11.0.13" 2021-10-19 LTS
```

## Download the CMAK and build
```
> cd ~
> curl -LO https://github.com/yahoo/CMAK/archive/refs/tags/3.0.0.5.tar.gz
> tar xvf 3.0.0.5.tar.gz
> rm -rf 3.0.0.5.tar.gz
> cd CMAK-3.0.0.5/

## 소스코드를 build하여 실행파일 생성
> ./sbt clean dist

## build를 통해서 생성된 실행파일(cmak-3.0.0.5.zip)을 확인한다.
> ls target/universal/
cmak-3.0.0.5.zip  scripts

## zip파일을 압축해제한다. (현재 디렉토리에 압축 해제)
> sudo yum install -y unzip
> unzip target/universal/cmak-3.0.0.5.zip
```

# STEP 2.CMAK 실행
```
## 현재 디렉토리에 압축해제된 디렉토리(cmak-3.0.0.5) 확인한다. 
> ls ~/CMAK-3.0.0.5/cmak-3.0.0.5
README.md  bin  conf  lib  share

## 모니터링 할 kakfka cluster의 zookeeper IP:PORT를 설정파일에 추가한다. 
> vi ~/CMAK-3.0.0.5/cmak-3.0.0.5/conf/application.conf
cmak.zkhosts="broker-01:2181"

> ~/CMAK-3.0.0.5/cmak-3.0.0.5/bin/cmak
```

# STEP 3. GCP 방화벽 허용 PORT 추가 
- CMAK에 접속하기 위한 9000 PORT 허용


# STEP 4. Web Browser로 접속
- http://<VM IP>:9000