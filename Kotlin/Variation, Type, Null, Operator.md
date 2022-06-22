# Variation, Type in Kotlin
- `var`: 가변
- `val`: 불변
- `val` 컬렉션에 요소 추가 가능
	- `java`에서 `final List<T>`에 요소를 추가할 수 있는 것과 같은 개념
- 모든 변수는 `val`을 우선으로 만들고 필요시에 `var`로 변경하자
- 코틀린에서는 `long`과 `Long`의 구분이 없음(불리언, 문자도 마찬가지)
	- 성능상 문제 없나?(박싱, 언박싱)
		- 보이는 것은 `Long` 즉, `Reference Type`이지만 연산시에는(상황에 딸) 코틀린이 알아서 `Primitive Type`으로 바꿔서 처리함
		- 개발자가 박싱과 언박싱을 고려하지 않아도 괜찮음
- 코틀린은 기본적으로 모든 변수가 `null`이 들어갈 수 없게끔 설계되어있음
	- `null`을 변수에 넣고 싶다면(`Reference Type`과 같이)
	- `var number: Long?`처럼 타입뒤에 물음표를 붙여줘야함
- 객체 인스턴스화를 할때 `new`키워드를 붙이면 안됨


---


# Null을 다루는법
- `null`이 반환될 수 있다면 반환타입뒤에도 물음표 붙여줘야함
- `null`이 가능한 타입을 완전히 다르게 취급

```kotlin
fun startsWithA3(str: String?): Boolean {
    return str.startsWith("A") // 컴파일 에러 발생
}
```
- `str`은 `null`이 들어올 수 있기 댸문에 `startsWith`메서드를 호출하려하면 컴파일 에러가 발생함

```kotlin
fun startsWithA3(str: String?): Boolean {
    if (str == null) {
        return false
    }
    return str.startsWith("A")
}

```
- 하지만 위와 같이 `null`체크를 하는 로직이 존재한다면 코틀린 `null`이 아니라고 추론해 `startsWith`메서드를 호출할 수 있게됨


## Safe Call, Elvis 연산자
- `Safe Call`: `null`이 아니면 실행, `null`이면 실행 x
```kotlin
val str: String? = "hello world"
println(str.length) // 불가능(컴파일 에러)
println(str?.length) // 가능
```

- `Elvis`연산자: 앞의 연산 결과가 `null`이면 뒤의 값 사용
```kotlin
val str2: String? = "ABC"
println(str2?.length ?:0) // 3출력
val str3: String? = null
println(str3?.length ?:0) // 0출력
```
	- `Elvis`연산자는 `Early Return`에도 활용가능
```kotlin
number ?: return 0 // number가 null이면 0을 리턴
```
	
## nullable type이지만 null이 절대될 수 없는 경우(단언)
```kotlin
fun startsWith(str: String?): Boolean {
    return str!!.startsWith("A") // 컴파일에러 x
}
```
- 만약에 `null`이 들어오면 `NPE`발생


## 플랫폼 타입
- `Kotlin`에서 `Java`코드를 가져다 사용해도 활용 가능
- `javax.annotation`, `org.jetbrains.annotation`
	- `@NotNull`, `@Nullable`등
- 플랫폼 타입: 코틀린이 `null`관련 정보를 알 수 없는 타입
	- 런타임시점에 `NPE`발생 가능
- `Java`코드를 사용할 때는 `null`체크를 꼼꼼히 해줘야함

### 1. @NotNull - 컴파일에러 x, 런타임에러 x
```java
public class Person {

    private final String name;

    public Person(String name) {
        this.name = name;
    }

    @NotNull
    public String getName() {
        return name;
    }
}

```
```kotlin
fun main() {
	val person = Person("철수")
	startsWith4(person.name)
}

fun startsWith4(str: String): Boolean {
    return str.startsWith("A")
}
```


### 2. @Nullable - 컴파일에러 o
```java
public class Person {

    private final String name;

    public Person(String name) {
        this.name = name;
    }

    @Nullable
    public String getName() {
        return name;
    }
}

```
```kotlin
fun main() {
	val person = Person("철수")
	startsWith4(person.name) // 컴파일 에러 발생!
}

fun startsWith4(str: String): Boolean {
    return str.startsWith("A")
}
```

### 3. 플랫폼타입 - 런타임에러 o
```java
public class Person {

    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

```
```kotlin
fun main() {
	val person = Person(null)
	startsWith4(person.name) // 런타임에러 발생!
}

fun startsWith4(str: String): Boolean {
    return str.startsWith("A")
}
```



---

# Type을 다루는법

## 기본타입
- `Kotlin`은 선언된 기본값을 보고 타입을 추론
- `Java`에서 기본 타입간의 변환은 암시적으로 이뤄질 수 있지만 `Kotlin`에서 기본 타입간의 변환은 명시적으로 이뤄져야함

```java
int number1 = 4;
long number2 = number1;

System.out.println(number1 + number2); // 가능!
```

```kotlin
val number1 = 4
val number2: Long = number1 // 컴파일에러(Type mismatch)
```

- `Kotlin`에서는 타입 변환을 위해 `to변환타입()`을 사용

```kotlin
val number1 = 4
val number2: Long = number1.toLong()

println(number1 + number2)
```


- `nullable type`이라면 적절한 처리가 필요 -> `Elvis`연산자와 같은


## 일반타입
```java
public static void printNameIfPerson(Object obj) {
	if (obj instanceOf Person) {
		Person person = (Person) obj;
		System.out.println(person.getAge();
	}
}
```

```kotlin
fun printNameIfPerson(obj: Any) {
    if (obj is Person) {
        val person = obj as Person // 생략가능
        println(person.name)
		  println(obj.name) // 가능!
    }
}
```
- `is` == `instanceOf`
- 만약 `obj`가 `Person`타입이라면 `Person`으로 변환
- `java`에서는 타입을 변환해줘야만하지만 `Kotlin`에서는 생략가능(스마트 캐스트)

- 해당 타입이 아니라면 이라는 의미로 `obj !is Person`이 가능
```kotlin
fun printNameIfPerson2(obj: Any) {
    if (obj !is Person) {
        println("person이 아니에요!")
    }
}
```


#### nullable 하다면?
`nullable type`의 변환을 위해서는 `as?`사용

```kotlin
fun printNameIfPerson3(obj: Any?) {
    val person = obj as? Person // Person?
    println(person.name) // 컴파일 에러 발생!
    println(person?.name)
}
```
- `as?`을 사용하게되면 `person`의 타입이 `Person?`이 됨(nullable)
- `obj`가 `null`이라면 `person`에 `null`이 들어감 -> `safe call`


## Any
- `Java`에서의 `Object` 역할
- 모든 객체의 최상위 타입
- 모든 `Primitive Type`의 최상위도 `Any`
- `Any`자체도 `null`을 담을 수 없음 -> `Any?`사용
- `Any`에 `equals`, `hashCode`, `toString`존재



## Unit
- `Java`의 `void`와 동일한 역할
- `void`와 다르게 `Unit`은 그 자체로 타입 인자가 될 수 있음
- 단 하나의 인스턴스만을 갖는 타입을 의미
	- 실제 존재하는 타입이라는 것을 표현
 


## Nothing
- 함수가 정상적으로 끝나지 않았다는 사실을 표현하는 역할
- 무조건 예외를 반환하는 함수, 무한 루프 등
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```


## 문자열
```kotlin
val person = Person("철수")
println("이름은 ${person.name}")
```
- `${변수}`를 사용해 포맷팅 가능

```kotlin
val str = """
    안녕하세요!
    Hello!
""".trimIndent()
```
- `java17`에 추가된 텍스트 블록과 같은 기능

```kotlin
var str2 = "ABC"
println(str2[0])
```
- 문자열에서 문자를 가져올때 배열을 사용할 수 있음


