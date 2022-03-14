# Spring for Apache Kafka 기본 2

## 1. Publish Message

### KafkaTemplate 설정
- `Spring Boot`를 사용하면 자동으로 만들어지는 객체
- 만약 직접 만드려면 `ProducerFactory`클래스를 이용해 생성해야함
	- 트랜잭션을 사용하지 않는 경우 싱글톤으로 생성
	- 트랜잭션을 사용하면 같은 `Producer`를 사용하는 다른 쓰레드에서 지연현상이 발생할 수 있음
		- 이를 위해 별도의 속성들을 제공(`ProducerPerThread`)

<br>

### KafkaTemplate 메시지 발송
- 기본적으로 비동기처리
	- 동기식으로 처리할 수 있지만 권장하지 않는 방법
- `Message<?>`객체를 이용
	- 메시지에 헤더로 정보 제공 가능
		- TOPIC, PARTITION_ID, MESSAGE_KEY, TIMESTAMP 등
- `ProducerRecord<K, V>`를 이용
- `topic`,  `partition`,  `offset`설정 후 전송

<br>

### RoutingKafkaTemplate
- 전송하는 토픽별로 옵션을 다르게 설정할 수 있음
	- 정규식으로 표현
- `transactions`, `execute`, `flush`, `metric` 커맨드를 지원 X

<br>

### ReplyingKafkaTemplate
- `Consumer`가 특정 데이터를 정상적으로 전달 받았는지 여부를 확인할 수 있음
- 3개의 헤더가 기본으로 정의됨
	- `KafkaHeaders.CORRELATION_ID`: 요청과 응답을 연결시킴
	- `KafkaHeaders.REPLY_TOPIC`: 응답 토픽
	- `KafkaHeaders.REPLY_PARTITION`: 응답 토픽의 파티션(optional)
- `AggregatingReplyingKafkaTemplate`: 여러 응답을 한번에 처리

<br>

### KafkaTemplate 직접 만들기
![스크린샷 2022-03-08 오후 6 06 22](https://user-images.githubusercontent.com/60773356/158139462-e53348af-d640-4bdc-b4a5-1908bb260c93.png)

- `DefaultKafkaProducerFactory`를 사용해 만들 수 있음
- `Map`으로 속성값에 대한 설정을 넣어주어야함
	- 부트스트랩 서버
	- `Serializer`

<br>

#### Auto Configuration에 의해 kafkaTemplate이라는 인스턴스가 자동으로 생성된다고 했는데 왜 동일한 이름으로 직접 만들어도 오류가 발생하지 않을까?
![스크린샷 2022-03-08 오후 6 33 58](https://user-images.githubusercontent.com/60773356/158139479-a686c92a-92af-446f-b73b-f101ce98edc2.png)

- `KafkaAutoConfiguration`을 살펴보면 `@ConditionalOnMissingBean`을 통해 `KafkaTemplate`타입의 빈이 정의가 되어있지 않을때만 자동생성 해주기 때문


<br>

### Async
![스크린샷 2022-03-08 오후 6 48 07](https://user-images.githubusercontent.com/60773356/158139502-16d9022d-23fb-42af-8cf4-cc29a1456141.png)

- `Producer`는 기본적으로 비동기 방식을 사용함
- 별도로 `Consumer`를 두지 않고 비동기 처리를 할 때 콜백 함수를 사용해 메시지가 정상적으로 전송되는지를 확인해보았음

![스크린샷 2022-03-08 오후 6 49 24](https://user-images.githubusercontent.com/60773356/158139516-67a29999-7a14-4111-94d8-50e930ad4f7e.png)

- 비동기 방식이기 때문에 임의로 `Thread.sleep`을 사용해 메시지가 출력되기 전에 서버가 내려가는 것을 막았음

![스크린샷 2022-03-08 오후 6 50 11](https://user-images.githubusercontent.com/60773356/158139539-56858bba-a297-4fcf-967e-7fd397fd0eb2.png)


<br>


### Sync
![스크린샷 2022-03-08 오후 6 53 36](https://user-images.githubusercontent.com/60773356/158139564-3a0ea047-0748-4cf6-82c3-906bf2b62b1b.png)


<br>

### RoutingKafkaTemplate 만들기
![스크린샷 2022-03-08 오후 7 00 55](https://user-images.githubusercontent.com/60773356/158139573-3fff617a-306c-425e-bac8-d4844a4e1eea.png)

- 여러 토픽에 대한 각자의 옵션을 지정해야 하므로 모든 타입을 받을 수 있도록 `Object` 타입을 사용해야함
- 정규표현식을 사용

<br>


---

<br>

## 2. Consume Message

`Record`는 하나씩 읽는 것이고, `Batch`는 리스트 형태의 `Record`를 가져와 한번에 처리하는 것

`Consumer`는 `Thread-Safe`하지 않으므로 `Listener`를 호출하는 쓰레드에서만 호출해야함

<br>

### Message Listener Container
- KafkaMessageListenerContainer
	- 싱글 쓰레드
- ConcurrentMessageListenerContainer
	- `KafkaMessageListenerContainer`를 1개 이상 사용하는 멀티 쓰레드
	- `start`, `stop`등의 메서드를 `foreach`로 순차적 실행

<br>

### @KafkaListener
- `@EnableKafka`를 사용하기 위해선 스프링 빈 중 이름이 `kafkaListenerContainerFactory`인 `ConcurrentMessageListenerContainer`객체 필요
- 스프링 부트에서는 이것이 자동으로 세팅됨
- 다양한 설정을 프로퍼티로 손쉽게 설정할 수 있음
- 메타 데이터 사용 가능
	- 헤더를 뽑아올 수 도 있고 `ConsumerRecordMetadata`를 이용해 수신할 수도 있음

<br>

### Payload Validator
- `KafkaListenerEndpointRegister`에 등록해 사용 가능
-  `LocalValidatorFactoryBean`을 사용하면  `javax.validation`에 정의된 유효성 검증을 할 수 있음

<br>

### Retrying Deliveries
- 기본적으로 에러가 발생하면 핸들러가 동작하지만 `RetryingMessageListenerAdapter`를 사용해 재시도 기능을 호출할 수 있음

<br>

### 메타 데이터
![스크린샷 2022-03-14 오후 4 40 23](https://user-images.githubusercontent.com/60773356/158139608-3d5466b2-6831-4383-bbe9-7177df2ddd30.png)

1. `Listener`에서 `ConsumerRecordMetadata`를 주입 받아 메타 데이터 사용
2. `Listener`에서 `@Header`를 사용하여 값을 주입 메타 데이터 사용

<br>
