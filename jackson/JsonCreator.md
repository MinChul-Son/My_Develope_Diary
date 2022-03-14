# `@JsonCreator`이란?

`@JsonCreator`는 `Deserialize`에 사용되는 애노테이션인데, 생성자 또는 팩터리 메서드를 컨트롤하기 위해 사용된다.

먼저, 아래 예제를 한번 살펴보자.
```java
@Getter
@ToString
@Setter
public class Person {
    private String name;
    private int age;
}
```

위와 같은 간단한 객체를 만들었고 이를 `ObjectMapper`를 통해 읽어보자.

```java
    @Test
    void Read_Json() throws JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        Person request = new Person();
        request.setName("민철");
        request.setAge(26);
        String json = objectMapper.writeValueAsString(request);

        Person person = objectMapper.readValue(json, Person.class);

        assertThat(person.getAge()).isEqualTo(26);
        assertThat(person.getName()).isEqualTo("민철");
    }
```

객체를 만들고 이를 `String`으로 읽어오고 다시 객체로 변환해보는 단순한 코드이다. 

당연하게도 테스트는 통과한다.


---

그럼 이건 어떨까?
```java
@Getter
@ToString
public class Person {

    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
이전과는 다르게 `@Setter`를 제거하였고 불변 객체가 되었다.

```java
    @Test
    void Read_Json() throws JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        Person request = new Person("민철", 26);
        String json = objectMapper.writeValueAsString(request);

        Person person = objectMapper.readValue(json, Person.class);

        assertThat(person.getAge()).isEqualTo(26);
        assertThat(person.getName()).isEqualTo("민철");
    }
```

이를 JSON 형식으로 표현해보면,

```json
{
    "name": "민철",
    "age": 26
}
```
과 같은 요청이 들어오고 있다고 볼 수 있을 것이다.


이 테스트는 성공일까 실패일까?

```
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.minchul.model.Person` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (String)"{"name":"민철","age":26}"; line: 1, column: 2]
```

위와 같은 예제를 뱉으며 테스트가 실패한다.

로그를 읽어보면 알 수 있듯이 `JSON`을 전달할 적절한 생성자를 찾지 못했다는 에러이다.


---

## 왜 이런 에러가 발생하는가?

원인을 알기 위해서는 먼저 `Jackson`이 `JSON`을 `Deserialize`하는 과정에 대해 알아봐야 한다.

우리가 `@JsonCreator`를 사용하여 변환기를 직접 만들지 않으면 `Jackson`은 다음과 같이 동작한다.
- 기본 생성자를 사용해 객체를 생성
- 필드가 `public`이면 직접 접근
- `private`라면 `setter`를 사용

하지만, 위의 예제 코드에는 직접 변환기를 만들지도 않았으며 기본 생성자, `setter`가 존재하지 않기 때문에 에라가 발생했던 것이다.

`@NoArgsConstructor + @Setter`의 조합을 대신하면서 객체의 불변성을 깨트리지 않게 도와주는 것이 바로 `JsonCreator`이다.

```java
@Getter
@ToString
public class Person {

    private final String name;
    private final int age;

    @JsonCreator
    public Person(@JsonProperty("name") String name, @JsonProperty("age") int age) {
        this.name = name;
        this.age = age;
    }
}
```

위와 같은 형태로 사용할 수 있다.

생성자 또는 팩토리 메서드 위에 애노테이션을 붙여 해당 생성자를 사용해 생성 시점에 필드를 채운다.

이 때 주의할 점은 `@JsonProperty(필드명)`을 필 수로 붙여줘야한다.

`Jackson`이 파싱하여 객체로 변환하는 과정에서 어떤 필드에 값을 채워넣어야하는지 모르기 때문이다.

추가로 만약 `JSON`이 매칭되지 않더라도 이 속성을 사용해 값을 우겨넣을 수 도 있다.

```java
@Getter
@ToString
public class Person {

    private final String name;
    private final int age;

    @JsonCreator
    public Person(@JsonProperty("theName") String name, @JsonProperty("age") int age) {
        this.name = name;
        this.age = age;
    }
}
```

```json
    "theName": "민철",
    "age": 26
```

위와 같이 필드와 다르게 `JSON`의 `Key`가 넘어온다고 하더라도 매칭시켜줄 수 있다.

해당 객체를 생성하는 생성자가 여러 개 일수도 있는데 이 때도 `Jackson`에게 어떤 생성자를 사용해야하는지 알려주는 용으로 사용할 수도 있다.

---

## 단점
- `Jackson`에 매우 의존적