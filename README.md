## 문서의 목적
본 문서는 KHELYS study에서 Apache Kafka를 학습하기 위해 작성되었습니다. (계속 작성 중입니다.)
주요 참고 문서는 다음과 같습니다.

http://kafka.apache.org/documentation.html

## Kafka 설치

``` wget http://mirror.apache-kr.org/kafka/0.8.2.1/kafka_2.10-0.8.2.1.tgz ```
``` tar xvzf kafka_2.10-0.8.2.1.tgz ```
``` ln -s kafka_2.10-0.8.2.1 kafka ```

### zookeeper 설정
``` vim config/zookeeper.properties ```

```
# 클라이언트가 접속할 포트
clientPort=2181
```

### zookeeper 구동
``` bin/zookeeper-server-start.sh config/zookeeper.properties ```

### kafka broker 설정
``` vim config/server.properties ```

필수 설정 요소
```
# 브로커의 ID. 각 브로커마다 유니크한 integer여야 한다.integer for each broker.
broker.id=0

# 소켓 서버가 listen할 포트
port=9092

# Zookeeper의 host:port를 지정

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.

zookeeper.connect=localhost:2181
```

그 외
```
# Hostname the broker will bind to. If not set, the server will bind to all interfaces
# ex) 설정하지 않는 경우, 모든 요청을 허용하며, localhost로 설정한 경우, localhost에서 온 요청만 허용한다.
#host.name=localhost

# 로그를 저장할 디렉토리. 콤마로 분리하여 설정하면 디렉토리 리스트도 가능
log.dirs=/tmp/kafka-logs
```

### kafka broker 구동
``` bin/kafka-server-start.sh config/server.properties ```

### topic 생성
``` bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test ```
--create 생성 작업
--zookeeper localhost:2181로 zookeeper 위치를 지정
--replication-factor로 토픽의 각 파티션에서 생성할 replication 수 지정
--partitions로 토픽의 파티션 수 지정
--topic으로 토픽 이름 지정


### topic list에서 확인
``` bin/kafka-topics.sh --list --zookeeper localhost:2181 ```

### kafka-console-producer 기동하여 메시지 생성
``` bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test ```
--broker-list에서 broker 지정. 10.0.0.1:9092,10.0.0.2:9092,10.0.0.3:9092 이런 식으로 여럿 지정이 가능
--topic으로 토픽 이름 지정

### kafka-console-consumer 기동하여 메시지 소비
``` bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning ```
--zookeeper로 Zookeeper 위치 지정
--topic으로 토픽 이름 지정
--from-beginning이 있는 경우, 해당 토픽의 제일 처음부터 읽으며, 없는 경우 새로 발생한 메시지만 소비



===

### replication을 생성하도록 테스트
broker 서버를 세 개 띄우기 위하여 설정
``` cp config/server.properties config/server2.properties ```
``` cp config/server.properties config/server3.properties ```

```
# 브로커의 ID. 각 브로커마다 유니크한 integer여야 한다.integer for each broker.
# 이 부분을 각각 1, 2로
broker.id=0

# 소켓 서버가 listen할 포트
# 이 부분도 충돌되지 않도록 각각 9093, 9094로
port=9092

# Zookeeper의 host:port를 지정
# 같은 Zookeeper를 이용하도록 지정

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.

zookeeper.connect=localhost:2181
```

### broker 서버의 구동을 시작
``` bin/kafka-server-start.sh config/server.properties ```
``` bin/kafka-server-start.sh config/server2.properties ```
``` bin/kafka-server-start.sh config/server3.properties ```

### topic 생성
``` bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic ```

### 생성된 topic 정보 확인
``` bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic ```
``` bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test ```

### Producer로 메시지 발행
``` bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic ```

### Consumer로 메시지 구독
``` bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic my-replicated-topic ```
