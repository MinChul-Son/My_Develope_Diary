# Ch01. Apache Kafka 기본 개념 및 이해

## 1. Apache Kafka
- 움직이는 데이터를 처리하기 위한 플랫폼
- Event Streaming Platform
	- 실시간 처리

<br>

### Event란?
- 비즈니스에서 일어나는 모든 일(데이터)을 의미
	- 웹 사이트에서 무언가를 클릭
	- 청구서 발행
	- 송금
	- 정산
	- 배송조회
- `Evnet`는 `BigData`의 특징을 가짐
	- 대용량

> `Event Stream`은 연속적인 많은 이벤트들의 흐름을 의미!!

<br>

### Kafka의 특징
1. 이벤트 스트림을 안전하게 전송(Publish & Subscribe)
2. 이벤트 스트림을 디스크에 저장(Write to Disk)
3. 이벤트 스트림을 분석 및 처리(Processing & Analysis

<br>

---

<br>


## 2. Topic, Producer, Consumer

### Topic
- `Kafka`안에서 메시지가 저장되는 장소(논리적인 표현)

<br>

### Producer
- 메시지를 만들어서 `Topic`으로 보내주는 역할을 하는 애플리케이션

<br>

### Consumer
- `Topic`내의 메시지를 가져와서 활용하는 역할을 하는 애플리케이션
- 하나의 `Consumer`는 하나의 `Consumer Group`에 포함되며, 그룹내의 `Consumer`들은 서로 협력해 `Topic`의 메시지를 분산 병렬 처리

<br>


> `Producer`와 `Consumer`는 서로 알지 못하고, 각각 고유의 속도로 `Commit Log`에 `Write`및 `Read`를 수행한다!!

> 다른 `Consumer Group`에 속한 `Consumer`들은 서로 관련이 없고 `Commit Log`에 있는 `Event`를 동시에 다른 위치에서 `Read`할 수 있다!

<br>

### Commit Log
- 추가만 가능하고 변경 불가능한 `Data Structure`로써 데이터는 항상 로그 끝에 추가되고 불변!
<img width="957" alt="스크린샷 2022-03-02 오후 9 55 29" src="https://user-images.githubusercontent.com/60773356/156569863-b5864180-1c66-4ed9-98be-9b9fe0784e86.png">

- Offset: `Commit Log`에서 `Event`의 위치(위 그림에서는 0~10까지의 offset 존재)
	- `CURRENT-OFFSET`: `Consumer`가 읽고 처리한 마지막 offset(어디까지 처리했는지 마킹)
	- `LOG-END-OFFSET`: `Producer`가 다음번에 `Write`할 offset
	- `Consumer Lag`: `CURRENT-OFFSET` 와 `LOG-END-OFFSET`의 차이

<br>

### Partition
- *Commit Log*가 바로 `Partition`
- 하나의 `Topic`의 하나 이상의 `Partition`으로 구성
- 병렬 처리를 위해 다수의 `Partition`사용
- `Partition`의 번호는 0번부터 시작해 오름차순
- `Topic`내의 `Partition`들은 서로 독립적
- 한번 저장된 데이터는 불변!
- 맨끝에만 추가되어 저장

<br>

### Segment
- 메시지가 저장되는 실제 물리 파일
- `Partition`에 실제 물리 파일인 `Segement`가 저장되어 있음
- `Segment File`이 지정된 크기보다 크거나 지정된 기간보다 오래되면 새 파일이 열리고 메시지는 새 파일에 추가됨


> `Topic` 생성시 `Partition`개수를 지정하고 각 `Partition`은 `Broker`에 분산되어 `Segment File`로 구성

<br>

---

<br>

## 3. Broker, Zookeeper

### Kafka Broker
- `Topic`과 `Partition`을 유지 및 관리
- `Partition`에 대한 `Read` 및 `Write`를 관리하는 SW
- `Kafka Server`라고 부르기도함
- `Topic`내의 `Partition`들을 분산, 유지 관리
- 각 `Broker`는 `ID`로 식별
- `Topic`의 일부 `Partition`을 포함
	- 만약, `Topic`에 4개의 `Partition`을 저장한다면 이 4개의 `Partition`은 각 `Broker`에게 분산되어 저장
- `Kafka Cluster`는 여러 `Broker`들로 구성
- `Client`에서는 특정 `Broker`에 연결하면 전체 `Kafka Cluster`에 연결됨
	- 하지만 특정 `Broker`장애를 대비해 전체 `Broker List`를 파라미터로 입력하길 권장
- 최소 3대 이상의 `Broker`를 하나의 `Cluster`로 구성해야하고 4대 이상을 권장하고 있음
- 각각의 `Broker`는 모든 `Broker`, `Topic`, `Partition`에 대해 알고 있음

<br>

### Bootstrap Servers
- `Broker Servers`를 의미
- 모든 `Kafka Broker`를 `Bootstrap`서버라고 부름

<br>

### Zookeeper
<img width="1134" alt="스크린샷 2022-03-03 오후 2 13 36" src="https://user-images.githubusercontent.com/60773356/156569952-16a889cd-9eca-4ad3-935c-b043c08bb6de.png">

- `Broker`를 관리하는 역할
- 변경사항이 존재할 때 `Kafka`에 알림을 보냄
- `Zookeeper`없이는 `Kafka`가 동작할 수 없다.
	- 2022년에 `Zookeeper`를 제거한 버전 출시 예정
- *`Apache Kafka 2.8.0`이후 버전부터는 `Zookeeper`가 제거되었음*
- 홀수의 서버로 작동하게 설계되어 있음 (최소 3, 권장 5)
- 분산형 `Configuration` 정보 유지, 분산 동기화 서비스를 제공해 대용량 분산 시스템을 위한 네이밍 레지스트리를 제공
- 분산 작업을 위해 `Tree`형태의 데이터 저장소!!
	- `Broker`들 간의 정보를 공유하고 동기화 등을 수행

<br>

### Zookeeper Failover
- `Quorum` 알고리즘 기반 [[쿼럼 알고리즘이란?](http://wiki.hash.kr/index.php/%EC%BF%BC%EB%9F%BC)
	- 의견을 도출해내기 위해 최소한(과반수)의 인원이 필요하다는 의미로 받아들이자
	- Why? 분산 환경에서 예기치 못한 장애가 발생할 때 시스템의 일관성을 유지하기 위해서!!
- `Ensemble`: `Zookeeper`서버의 클러스터
	- 만약 `Ensemble`이 3대의 `Zookeeper`가 구성되어있다면 1대가 장애로 인해 서버가 다운되더라도 `Quorum`(정족수)인 2 이상이기 때문에 정상동작
	- 1대가 추가로 다운되어 1대만 남았다면 정족수인 2 미만이므로 동작 X

<br>

---

<br>

## 4. Producer
*메시지를 생산해서 `Topic`으로 메시지를 보내는 애플리케이션!*

<br>

### Record(Message)구조
<img width="1018" alt="스크린샷 2022-03-03 오후 2 59 10" src="https://user-images.githubusercontent.com/60773356/156569984-f5a1a027-1fcb-4d2c-90e0-11ad4f7ecf92.png">- `Header`, `Key`, `Value`로 구성되어 있음

- `Message` == `Record` == `Event` == `Data`
- `Key`, `Value`는 `Avro`,  `JSON`과 같은 다양한 형태가 가능

<br>

### Serializer / Deserializer
**`Kafka`는 `Record`를 `Byte Array`로 저장!**

**⚙️동작 과정**
- `JSON`, `String`, `Avro`등 데이터를 원하는 포맷으로 변환 후 `Record`의 `Key`나 `Value`에 담아 전송
- `Producer` 내부의 `Serializer`가 이를 `Byte Array`로 변환
- `Consumer`는 `Byte Array`를 `Deserializer`를 통해 다시 원본  `Record`로 변환하여 활용

<br>

**`Key`와 `Value`용 `Serializer`/ `Deserializer`를 각각 설정해야한다.**

```
private Properties props = new Properties();

props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG. "broker101:9092, broker102:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, org.apache.kafka.common.serialization.StringSerializer.class);
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, io.confluent.kafka.serializers.KafkaAvroSerializer.class);
```

<br>

### Kafka Producing Architecture
<img width="1192" alt="스크린샷 2022-03-03 오후 3 12 15" src="https://user-images.githubusercontent.com/60773356/156570003-38bee87b-df50-4841-949a-4fa633c54e78.png">

<br>

### Partitioner
<img width="1296" alt="스크린샷 2022-03-03 오후 3 15 01" src="https://user-images.githubusercontent.com/60773356/156570021-d75bdb44-a6f5-4020-8f80-6b83ec4cb772.png">

- 메시지를 `Topic`의 어떤 `Partition`으로 보낼지 결정
- 내부적으로 해시 알고리즘을 기반으로 계산
- 메시지의 `Key`는 반드시 `null`이 아니어야함!!

**만약 `Key`가 `null`이면요?**
- `2.4`버전 이전에는 `Key`가 `null`이면 `Round Robin Policy`을 사용
- `2.4`버전 이후에는 `Sticky Policy`를 사용
	- 랜덤으로 `Partition`을 선택하고 그 `Partition`의 `Batch`가 닫힐 때까지 동일한 `Partiton`으로 전송

<br>

`Key`가 `null`인 6개의 메시지가 있다고 가정해보자.

`Round Robin`을 쓸 때는 `Broker`가 1개씩 6개의 `Partition`에게 전송해야하므로 6번의 전송이 일어나지만 `Sticky`는 각 `Partition`의 `Batch Size`만큼 전송할 수 있기 때문에 좀 더 효율적이다.
 
-> `Batch Size`가 3이라면 두번의 전송만 일어난다는 의미임

<br>

---

<br>

## 5. Consumer

<br>

### Consuming from Kafka
- `Partition`으로부터 `Record`를 가져옴 -> *Poll*
- `Consumer`는 각각 고유의 속도로 `Commit Log` 순서대로 `Record`를 읽음

<br>

### Consumer Offset
- `Consumer`가 자동이나 수동으로 읽은 데이터를 중복으로 읽는 것을 방지하기 위해 존재
	- 읽은 위치를 `commit`해 저장
- `Topic __consumer_offsets`라는 토픽에 저장됨
- 만약, 여러개의 `Partition`이 존재하고 `Consumer`는 하나만 존재한다면?
	- 각 `Partition`별 `offset`을 모두 저장함
- `Partition`의 개수와 `Consumer Group`내의 `Consumer`개수가 일치한다면 1:1로 매핑됨

<br>

### Consumer Group
- 동일한 `group.id`를 가진 `Consumer`는 같은 `Consumer Group`에 속함
- 동일한 `Topic`을 사용하는 여러 그룹이 존재할 수 있음
- 하나의 그룹에 속하는 `Consumer`들의 작업량을 어느정도 균등하게 분할함

<br>

---

### Message Ordering
- `Partition`이 2개 이상인 경우 모든 메시지에 대한 전체 순서 보장이 불가능
	- `Partition`별로 독립적으로 동작하기 때문
- `Partition`을 1개로 구성하면 전체 순서 보장은 가능하지만 처리량이 저하
	- 대부분은 전체 순서 보장보다는 동일한 `Partition`내에서 순서를 보장하는 로직임
- 동일한 `Key`를 가진 메시지는 동일한 `Partition`에만 전달되어 `Key`레벨의 순서 보장이 가능해짐
	- 쿠폰 발급을 원하는 메시지는 쿠폰 발급 내에서 순서 보장을 하고, 주문은 주문끼리 순서 보장을 한다는 뜻으로 이해함
	- 11번가 전체 메시지에 대한 순서보장은 필요없으니까! (맞나?🤔)
- 운영중에 `Partition`개수가 변경되면 순서 보장이 아예 불가능해짐

<br>

### Cardinality
- 특정 데이터 집합에서 유니크한 값의 개수
- `Key Cardinality`는 `Consumer Group`의 개별 `Consumer`가 수행하는 작업의 양에 영향
	- 특정 `Partition`에만 데이터가 쌓을 수 있음
- `Key`선택이 잘못되면 작업 부하가 고르지 않게됨
- `Key`는 `Integer`, `String`과 같이 단순 유형이 아닌 `Json`, `Avro`같은 복잡한 객체일 수 있음
- **모든 `Partiton`에 `Record`를 고르게 배분하는 `Key`를 만드는 것이 매우 중요함!!**

<br>

### Consumer Failure
- `Consumer`에 장애가 발생하면 해당 `Consumer`(장애가 발생한)가 `Consume`하고 있던 `Partition`을 `Rebalancing`과정을 통해 다른 `Consumer`가 `Consume`할 수 있게 함

<br>
<br>

---

## 6. Replication

### Broker에 장애가 발생하면?
- 해당 `Broker`내의 `Partiton`을 `Producer`와 `Consumer`가 모두 사용할 수 없게되는 문제 발생
- 다른 `Broker`에서 장애가 발생한 `Partition`을 새로만들면 안되나요?
	- 내부에 쌓여있던 기존 메시지와 `offset`정보는?
	- 미리 복제해놓자!
- **`Replication`은 `Partition`을 미리 다른 `Broker`상에 복제해 장애를 미리 대비하는 기술!!**
	- 원본이 있는 `Broker` : `Leader`
	- 복제본이 있는 `Broker` : `Follower`
	- 이 때 `Producer`는 `Leader`에 `write`하고 `Consumer`는 `Leader`에서 `read`함
	- `Follower`는 `Leader`의 `Commit Log`에서 데이터를 `Fetch Reqeust`로 복제
	- `2.4`버전부터 `Follower`의 복제본도 `read`할 수 있는 옵션 추가
- `Leader`에 장애가 발생하면 `Follower`중에서 새로운 `Leader`를 선출하고 정상적으로 동작

<br>

### Partition Leader에 대한 자동 분산
- 하나의 `Broker`에만 `Leader`들이 몰려있으면 부하가 집중됨(Hot Spot)
- 분산을 위한 옵션
	- `auto.leader.rebalance.enable` :기본값 true
	- `leader.imbalance.check.interval.seconds`: 기본값 300 sec
		- 300초마다 불균형을 체크

<br>

### Rack Awareness
- `Rack`에 분산하여 장애대비
- 복제본을 `Rack`간의 균형을 유지하며 분산시킴
