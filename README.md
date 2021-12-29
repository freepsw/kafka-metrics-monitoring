# Understanding the metrics of apache kafka (broker, producer, consumer) and implementing the monitoring system
- Apache kafka의 주요 metrics 정보를 이해하고, 
- 이러한 metrics 들을 다양한 방법으로 수집 및 모니터링 할 수 있는 방안 
- Fastcampus 강의에서 사용한 실습코드로 구성함.

## Directory layout
### Code 

- code : 실습에 사용되는 script 및 코드 관리 디렉토리
    - ch.03 : Chapter 03. Apache Kafka 모니터링 시스템 구축
        - 02. GCP에서 Kafka 구성하기
        - 03. 실습용 Producer 구성
        - 04. 실습용 Consumer 구성
        - 05. Console 기반 모니터링 실습
        - 06. 오픈소스 기반 모니터링 실습 (CMAK)
        - 07. 오픈소스 기반 모니터링 실습 (AKHQ)
    - ch.04 : Chapter 04. Apache Kafka 모니터링 시스템 실습 
        - 01. Elastic Stack기반 모니터링 실습 (Elastic Stack 설치)
        - 03. Elastic Stack기반 모니터링 실습 (MetricBeats활용)
        - 04. 클라우드 서비스(DataDog)을 활용한 모니터링 실습
    - ch.05 : Chapter 05.  Apache Kafka 모니터링 활용 (성능 최적화) 
        - 01. 처리성능 개선을 위한 Kafka 설정 및 시긱화
        - 02. 전송 지연 최소화를 위한 Kafka 설정 및 시각화
        - 03. 서비스 가용성을 위한 Kafka 설정 및 시각화

- data : 실습에 사용한 데이터

### 실습 환경
- Google GCP에서 생성한 Linux 서버(CentOS7) 기반의 명령어 및 실행 환경으로 구성