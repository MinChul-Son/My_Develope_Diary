# yml에서 변수를 리스트 형식으로 사용하는 법

로컬환경, 개발환경, 상용환경으로 환경을 분리할 때, `yml`에 변수를 넣어두고 코드에서 불러와 사용하는 방법을 흔히 사용하곤 한다.

```
person:
  name: '민철'
```
이런 방식으로 말이다.

사용하는 쪽에서는 아래와 같다.

```
@Value("${person.name:})
private String name;
```

---

추가적으로 `yml`은 `List`나 `Map`과 같은 다양한 데이터 유형을 지원하고 있다.

```
programming:
  language:
    - C
    - Java
    - Python
    - JavaScript
```

이를 가져오려면

```
@Value("${programming.language[0]:}")
private String language; // C

@Value("${programming.language[1]:}")
private String language1; // Java

@Value("${programming.language[2]:}")
private String language2; // Python

@Value("${programming.language[3]:}")
private String language3; // JavaScript
```
이와 같이 사용할 수 있다.

---

한 단계 더 가면 객체나 객체 리스트로 바인딩 또한 가능하다.

```
korea:
  people:
    -
      name: '민철'
      age: 26
    -
      name: '철수'
      age: 13
    -
      name: '스폰지밥'
      age: 30
```

위와 같은 형태로 `yml`에 정의해준다.

`-` 하위에 있는 3개의 사람은 `List`에 담길 원소를 의미한다.

이제 바인딩 시켜보자.

```java
@ToString
@Setter
@Getter
@Component
@ConfigurationProperties(prefix = "korea")
public class PeopleProps {

    private List<Person> people;

    record Person(String name, long age) {
    }

}
```

여기에는 `@Value`를 사용한 `SpEl'은 사용할 수 없고 `setter` 또는 적절한 생성자를 매핑하여 바인딩 시켜주면 된다.

`setter` 또는 생성자가 필요하기 때문에 변수명이 `yml`에 선언된 것과 정확히 일치해야한다는 것을 주의하자.

또한, `@Component` 또는 `@Configuration`클래스 내에서 `@Bean`을 통해 스프링 빈으로 등록해야만한다.









