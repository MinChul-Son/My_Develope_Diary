# `직렬화`와 `역직렬화`

## 왜 필요한가?

먼저, 직렬화와 역직렬화가 필요한 배경을 이야기해보자.

이를 위해선 객체가 저장되는 과정에 대한 사전 지식이 필요하다.

`java`개발자라면 알다시피 객체, `Object`는 메모리의 `Heap`영역에 저장된다. 그리고 그 저장된 `Heap`영역의 주소가 인스턴스에 담기게되는 것이다.

```java
Example example = new Example();
```
`example`이라는 **객체 인스턴스에는 실제 값이 담겨있는 것이 아닌 `Heap`영역의 주소값이 담겨있다.**

만약 이 객체를 통신을 위해 주고받아야 한다면? 각 PC마다 각자의 가상 메모리를 가지고 있는데 이 주소값이 의미가 있을까?

따라서 주소값이 아닌 `Byte`형태로 컴퓨터가 이해할 수 있는 바이트 코드로 변환하여 전달하고 이를 다시 객체로 변환하는 `직렬화`, `역직렬화`가 필요한 것이다.

![image](https://user-images.githubusercontent.com/60773356/134905773-cf0c6d02-eddf-457c-9f92-88755c6caf0c.png)

## 사용
`java.io.Serializable` 인터페이스를 구현하여 직렬화, 역직렬화를 사용할 수 있다.

직렬화를 위해서는 `java.io.ObjectOutputStream`를 사용한다.

```java
@Entity
@AllArgsConstructor
@toString
public class Post implements Serializable {
  private static final long serialVersionUID = 1L;
    
  private String title;
  private String content;
  
  Post post = new Post("제목", "내용");
  byte[] serializedPost;
  try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
    try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
        oos.writeObject(post);

        serializedPost = baos.toByteArray();
    }
}
```

역직렬화에는 `java.io.ObjectInputStream`을 사용한다.

```java
try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedPost)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {

        Object objectPost = ois.readObject();
        Post post = (Post) objectPost;
    }
}
```

## 주의할 점
* 자바 직렬화 대상 객체는 동일한 `serialVersionUID` 를 가지고 있어야하며 개발자가 직접 관리하는 것이 좋다.
* 클래스의 변경을 예측할 수 없거나, 자주 변경되는 클래스는 직렬화 사용을 지양하자.
* 역직렬화 구현시에 실패하는 상황에 대한 예외처리를 해야한다.
* 직렬화 데이터는 `JSON`포멧을 사용하는 것이 2~10배 효율적이다.

### 참고
[우아한형제들 테크 블로그](https://techblog.woowahan.com/2550/)
