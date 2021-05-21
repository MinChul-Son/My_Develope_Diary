# 실패 테스트에서의 예외처리를 Junit5에서는 어떻게 처리할까??

## 잠깐! Junit4에서는 어떻게 했더라??
### 상황 : 회원 등록 시에 똑같은 이름이 존재한다면, 중복 회원 예외 발생!
```java
@Test(expected = IllegalStateException.class)
public void 중복회원_예외() {
    Member member1 = new Member();
    member1.setName("son");

    Member member2 = new Member();
    member2.setName("son");

    memberService.join(member1);
    memberService.join(member2);

	Assert.fail("예외가 발생해야 한다."); // 이 메서드가 실행되면 테스트는 실패!
    }
```
* expected 속성을 사용하여 어떤 예외가 발생할지를 미리 명시하여 테스트를 진행하였다.

## Junit5에서는??
### 상황 : 위와 같음
```java
@Test
public void 중복회원_예외() {
    Member member1 = new Member();
    member1.setName("son");
    
    Member member2 = new Member();
    member2.setName("son");

    memberService.join(member1);
    assertThrows(IllegalStateException.class, () -> {
        memberService.join(member2);
    });
```
* SpringBoot가 현재 사용하는 테스트 유닛은 Junit5이다.
* Junit5에서는 테스트 애노테이션에 expected속성을 사용할 수 없다.
* 대신, assertThrows를 사용한다.
  - 람다식 형태로 실행문을 내부로 던져준다.
  - 이때, 발생할 예외를 앞에서 클래스 형식으로 작성해줌.

* 만약 명시한 예외가 발생하지 않는다면 테스트는 실패한다.
