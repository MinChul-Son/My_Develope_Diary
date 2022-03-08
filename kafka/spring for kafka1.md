# Spring for Apache Kafka 기본
## 1. Quick Start

<br>

### 카프카 실행
- brew services start zookeeper
- brew services start kafka
- 기본적으로 카프카는 9092, 주키퍼는 2181포트를 사용함

<br>

### 토픽 생성
- kafka-topics --create --topic quickstart-events --bootstrap-server localhost:9092
	- `quickstart-events`라는 이름으로 토픽을 9092번 포트를 사용하는 부트스트랩 서버에 생성하는 명령어

<br>

### 메시지 전송
![스크린샷 2022-03-07 오후 5 28 41](https://user-images.githubusercontent.com/60773356/157157632-fd226968-0516-4814-b3ce-00997a3f53a7.png)

- 간단한 메시지를 전송
- 스프링 빈 형태로 등록해주어야하는데, `KafkaTemplate`을 사용함
	- `KafkaTemplate<String, String>`은 `key`와 `value`가 모두 `String`이라는 의미
- `send`를 통해 앞서 생성한 `topic`으로 데이터를 전송

<br>

### Consume
![스크린샷 2022-03-07 오후 5 30 58](https://user-images.githubusercontent.com/60773356/157157644-6ee899f5-cfbc-4b6a-8af2-c7b231acffdf.png)

- 마찬가지로 스프링 빈 형태로 등록해주어야함
- `@KafkaListener`애노테이션을 사용하면 `Consumer`로 등록됨
	- `id`속성은 그룹 아이디를 의미하고 `topics`가 어떤 토픽을 사용할지에 대한 속성
- 전송한 데이터가 `String`이기 때문에 파라미터로 `String`을 받음


![스크린샷 2022-03-07 오후 5 34 45](https://user-images.githubusercontent.com/60773356/157157672-45fd2b53-ee2d-421b-9b4a-c0df5d0bfd04.png)

- 정상적으로 메시지가 출력됨을 확인할 수 있음

<br>
<br>

---

<br>

## 2. Kafka Topic

<br>

### 토픽을 생성할 때 고려할 점
- 토픽명 
	- 토픽명은 한번 정하면 바꾸기가 매우 어렵기 때문에 규칙을 정해 패턴화하여 의미를 부여하는 것이 좋음
- 토픽의 파티션 개수 계산
	- 1초당 메시지 발생 수 / `Consumer Thread`가 1초당 처리하는 메시지의 수
	- ex: 1000개 / 100개 = 10개의 파티션 필요
	- 파티션의 수를 늘릴 수는 있지만 줄일 수는 없음
	- 파티션 수가 증가하면 쓰레드 수가 증가하므로 비용이 증가함(더 많은 서버 필요)
- Retention 시간
	- `Broker`가 메시지를 얼마나 보관할지를 설정하는 것
	- 시간 또는 바이트 크기를 초과했을 때 자동으로 지우게 할 수 있음
	- 디스크 크기와 데이터의 중요성에 따라 판단

<br>

### TopicBuilder를 사용한 토픽 생성
![스크린샷 2022-03-08 오전 10 53 04](https://user-images.githubusercontent.com/60773356/157157691-e09ef915-263f-4bb6-ae68-18b8b1b5d087.png)

- 스프링 빈으로 등록시켜줘야함
- `TopicBuilder`를 사용해 생성하는데 반환 타입은 `NewTopic`


토픽 리스트 확인을 위해 `kafka-topics --list --bootstrap-server localhost:9092`명령어로 확인을 해보면 아래와 같이 잘 생성된 것을 확인할 수 있음

![스크린샷 2022-03-08 오전 10 54 09](https://user-images.githubusercontent.com/60773356/157157710-012119e9-7ed5-4750-a579-b4c011033fe4.png)



또한, 한번에 여러개의 토픽을 생성할 수 있도록 `NewTopics`를 지원해주고 있음

![스크린샷 2022-03-08 오전 11 05 40](https://user-images.githubusercontent.com/60773356/157157732-b85614e5-8193-4879-badd-2029b8dd2f59.png)


![스크린샷 2022-03-08 오전 11 06 59](https://user-images.githubusercontent.com/60773356/157157744-686851ba-eb2a-491e-8c1c-a3cd6d433146.png)
- `TopicBuilder`는 해당 토픽에 대한 옵션을 설정할 수 있음
- 파티션의 개수, 복제본의 개수 등 여러 옵션을 지정해줄 수 있다.


![스크린샷 2022-03-08 오전 11 08 02](https://user-images.githubusercontent.com/60773356/157157787-0829e087-81fc-4e6a-ac34-7768043ca62b.png)

- config에 대한 옵션은 이미 정적 필드로 만들어져 있기 때문에 필요한 설정을 가져다 사용하면 됨

<br>

### NewTopics의 동작 원리
![스크린샷 2022-03-08 오전 11 12 23](https://user-images.githubusercontent.com/60773356/157157818-75136fdd-9154-40bc-9e80-e6327c961c86.png)

- `KafkaAdmin`이라는 객체가 초기화되면서 토픽이 생성됨
- 이는 `Spring`에서 `AutoConfig`로 자동으로 만들어지도록 되어있기 때문에 스프링 컨텍스트가 띄워질 때 아래의 `initialize()`메서드가 실행되면서 토픽이 생성되는 구조
- `KafkaAdmin`의 `autoCreate`라는 프로퍼티를 `false`로 지정하게되면 자동 생성되지 않음(디폴트가 `true`)
- 더 자세한 내용은 내부의 메서드를 살펴보자

<br>

### AdminClient
![스크린샷 2022-03-08 오전 11 20 02](https://user-images.githubusercontent.com/60773356/157157848-3ff72139-01db-4ca9-958c-0f057b547af2.png)

- 토픽, 브로커, 설정 등을 관리하는 역할을 하는 객체
- 별도로 구현할 필요없이 이미 만들어져있는 기본 `AdminClient`를 스프링 빈으로 등록하면 사용할 수 있음



![스크린샷 2022-03-08 오전 11 23 34](https://user-images.githubusercontent.com/60773356/157157874-17c45c6d-5f8a-449f-b9ff-065ef47a91f0.png)

- 등록한 `AdminClient`를 가져와 토픽 정보를 출력해보는 코드
- 내부에는 `Map`으로 토픽 정보가 저장되어 있음
	- `Key`는 토픽명, `Value`는 `TopicListing`이라는 객체


`TopicListing`이라는 객체는 토픽명과 `internal` 정보를 가지고 있는 단순한 객체임

![스크린샷 2022-03-08 오전 11 25 19](https://user-images.githubusercontent.com/60773356/157157883-88e9830e-b3db-4e11-af0b-f40dd69f1eb5.png)


![스크린샷 2022-03-08 오전 11 26 01](https://user-images.githubusercontent.com/60773356/157157890-aa909c52-24a1-4923-b14f-b051bdde29af.png)

- 여기서 `internal`의 의미는 해당 토픽이 사용자가 외부에서 만든 토픽이 아닌 카프카 내부에서 생성된 토픽인가를 의미
	- 예를들어, 모든 토픽의 `offset`정보를 담고 있는 `__consumer_offsets`




해당 토픽에 대한 더 자세한 정보는 `describeTopics`메서드를 사용하여 가져올 수 있음

![스크린샷 2022-03-08 오전 11 30 01](https://user-images.githubusercontent.com/60773356/157157900-d9cd6253-441c-491e-b511-76ad56a42840.png)
- 이 때 파라미터로 컬렉션 객체에 토핑명을 담아서 넘겨줘야함

![스크린샷 2022-03-08 오전 11 30 29](https://user-images.githubusercontent.com/60773356/157157917-4f6d3209-14eb-40bc-bbcf-ed7e804a856e.png)



`AdminClient`를 사용해 토픽을 삭제할 수도 있음
`adminClient.deleteTopics(Collections.singleton(topicName));`
- 마찬가지로 컬렉션 객체에 토픽명을 담아 넘겨주면됨
	- 여러 토픽명을 넘기면 여러개의 토픽을 한번에 삭제할 수도 있음




