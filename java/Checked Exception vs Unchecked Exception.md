# `Checked Exception` vs `Unchecked Exception`
![image](https://user-images.githubusercontent.com/60773356/134798844-c1c3b61c-b1d1-44a5-8b03-dca47749bf67.png)

간단히 얘기하자면, `RuntimeException`을 상속한 예외는 `Unchecked Exception`이고 그렇지 않은 예외는 `Checked Exception`이다.

이름에서 추측할 수 있듯이 `Unchecked Exception`은 체크를 하지 않아도 되는 예외이다. 즉, **명시적으로 예외처리를 하지 않아도 된다.**

그에 반해 `Checked Exception`은 `try-catch`과 같은 예외 처리를 필수로 해야한다.

흔히, 개발자들이 `Custom Exception`을 만들 때 `RuntimeException`을 상속하여 구현하는데 명시적으로 예외처리를 하지 않고 예외를 전환시키는 방식을 사용하는 것이다.

아래와 같은 방식으로 사용하곤 한다.

```java
public class DuplicatedException extends RuntimeException{
    public DuplicatedException(String message) {
        super(message);
    }
}

--------------------------------
public void checkDuplicatedEmail(final String email) {
  if (memberRepository.existsByEmail(email)) {
    return true;
  }
  throw new DuplicatedException("이미 존재하는 이메일이에요!@!@!@);
}
```

더 자세한 차이점들을 알아보자.

### `Checked Exception`
* 예외 확인 시점 : 컴파일 단계
* 예외 처리 여부 : 반드시 처리해야함
* 예외 발생시 트랜잭션 : 롤백하지 않음
* 종류 : `RuntimeException`을 상속하지 않는 모든 예외
  * ex : `IOException`, `SQLException`, `ClassNotFoundException` .....


### `Unchecked Exception`
* 예외 확인 시점 : 런타임 시점
* 예외 처리 여부 : 명시적으로 처리하지 않아도 괜찮음
* 예외 발생시 트랜잭션 : 롤백
* 종류 : `RuntimeException`을 상속하는 모든 예외
  * ex : `NullPointerException`, `IndexOutOfBoundException`, `ClassCastException` .....



### 참고
[Checked and Unchecked Exceptions in Java](https://www.baeldung.com/java-checked-unchecked-exceptions)
