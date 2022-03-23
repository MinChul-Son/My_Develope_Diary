# `@EnableKafka`를 왜 붙이는 걸까?


![스크린샷 2022-03-23 오후 11 04 05](https://user-images.githubusercontent.com/60773356/159717639-fd421003-85c0-48c4-a55d-cded0aec0617.png)

`Spring Kafka`를 통해 간단한 실습을 하며 `Consumer`를 위한 `KafkaListenerContainerFactory`를 스프링 빈으로 등록하여 도메인 객체를 메시지로 주고 받게 설정하였다.

이 코드는 아무런 문제 없이 동작한다.


`KafkaListenerContainerFactory`에 대한 추가적인 공부를 위해 공식 문서를 찾으려 구글링을 통해 읽어 보고 있었는데, 연관 검색으로 나온 수많은 블로그 글 들에서는 
`KafkaListenerContainerFactory`를 위한 설정 클래스에서는 `@EnableKafka`를 필수적으로 붙어야한다는 글들이 정말 많았다.

앞서 말했듯이 내 코드는 정상 동작하는데, `@EnableKafka`가 필수라는 말은 반은 맞고 반을 틀리다.

정답은 `KafkaAutoConfiguration`에 있다. - [공식 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/kafka/KafkaAutoConfiguration.html)

이름 그대로 `Kafka`에 대한 기본 설정 및 구성을 자동으로 해주는 역할을 한다.

이는 `Spring Boot`를 사용한다면 사용자가 수동으로 제외하지 않는한 동작하는데, `KafkaTemplate<String, String>`을 따로 만들어주지 않아도 만들어져있는 것 또한 이 클래스가 동작했기 때문이다.

코드를 직접 따라가보자.

![스크린샷 2022-03-23 오후 11 23 27](https://user-images.githubusercontent.com/60773356/159721716-b89d184a-0c37-4506-bf63-b5a4e1e61e11.png)

`KafkaAutoConfiguration`의 코드 일부이다.

클래스 레벨에 붙은 애노테이션을 보면 `@Import({ KafkaAnnotationDrivenConfiguration.class, KafkaStreamsAnnotationDrivenConfiguration.class })`가 있는데,

여기서 `KafkaAnnotationDrivenConfiguration`를 따라가보자.

내부에 수많은 기본적으로 생성되는 빈이 존재하지만 또 다시 클래스 레벨에 붙은 애노테이션을 보면

![스크린샷 2022-03-23 오후 11 26 27](https://user-images.githubusercontent.com/60773356/159722268-2c20cba7-d5f0-4148-9d9e-360bf6ff4323.png)

`@ConditionalOnClass(EnableKafka.class)`를 확인할 수 있다.

즉, `KafkaAutoConfiguration`가 `@EnableKafka`를 위한 설정을 기본적으로 해주고 있다.

만약 `Spring Boot`를 사용하지 않고 `Spring`만 사용한다면 많은 블로그 글들처럼 `@EnableKafka`를 붙여줘야하는 것이 맞다.

**하지만, `Spring Boot`를 사용한다면 필요하지 않다.**

<br>

대부분의 블로그들은 `Spring Boot`를 사용하고 있음에도 무조건 적으로 붙여야한다고 설명하고 있다.

역시 공식 문서를 먼저 참조하여 코드를 뜯어본 후에, 정리된 블로그 글을 보며 내가 이해한 것과 다른 사람이 이해한 것에 어떤 차이가 있는지를 확인하는 것이 개발 공부에 있어 국룰(?)이라는 생각이 든다.
