# `@DirtiesContext`

[레퍼런스](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-dirtiescontext)

`baeldung`에서 가이드를 찾아보다가 처음보는 `@DirtiesContext`라는 애노테이션을 만났는데 어떨 때 사용하고 무엇을 하는 애노테이션일까?

## `ApplicationContext`
`Spring`에서 테스트를 작성하다보면 각 개별 테스트는 통과하지만 모든 테스트를 실행시키면 테스트가 실패하는 경우가 생기게 되는데

이는 `ApplicationContext`가 테스트 과정에서 더러워졌기 때문이다.
- 스프링 빈의 상태가 변경되는 등 테스트가 다른 테스트에 영향을 줄 수 있는 상황

이 상황에서 `ApplicationContext`를 새로 빌드해 초기화시키는 역할을 해주는 애노테이션이다.

`Spring Test`에서는 같은 `SpringBootApplication`을 이용하기 때문에 `ApplicationContext`를 재활용하기 때문에 위와 같은 문제의 상황이 발생할 수 있다.

## How to use?
`@DirtiesContext`는 메서드나 클래스 레벨에 붙일 수 있다.

### 클래스 레벨
```java
@DirtiesContext(classMode = BEFORE_CLASS) 
class FreshContextTests {
    // some tests that require a new Spring container
}
```
- 현재 테스트 클래스 이전에 `ApplicationContext`를 새로 빌드한다.

<br>

```java
@DirtiesContext 
class ContextDirtyingTests {
    // some tests that result in the Spring container being dirtied
}
```
- 현재 테스트 클래스 이후에 `ApplicationContext`를 새로 빌드한다.
- 클래스 레벨에 붙이면 기본값

<br>

```java
@DirtiesContext(classMode = BEFORE_EACH_TEST_METHOD) 
class FreshContextTests {
    // some tests that require a new Spring container
}
```
- 현재 테스트 클래스의 각 테스트 메서드 이전에 `ApplicationContext`를 새로 빌드한다.


<br>

```java
@DirtiesContext(classMode = AFTER_EACH_TEST_METHOD) 
class ContextDirtyingTests {
    // some tests that result in the Spring container being dirtied
}
```
- 현재 테스트 클래스의 각 테스트 메서드 이후에 `ApplicationContext`를 새로 빌드한다.


<br>

### 메서드 레벨
```java
@DirtiesContext(methodMode = BEFORE_METHOD) 
@Test
void testProcessWhichRequiresFreshAppCtx() {
    // some logic that requires a new Spring container
}
```
- 현재 테스트 메서드 이전의 `ApplicationContext`를 새로 빌드한다.

<br>

```java
@DirtiesContext 
@Test
void testProcessWhichDirtiesAppCtx() {
    // some logic that results in the Spring container being dirtied
}
```
- 현재 테스트 메서드 이후의 `ApplicationContext`를 새로 빌드한다.
