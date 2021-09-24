# Session Clustering

한마디로 여러 개의 `WAS`의 세션을 공유하는 기술이다.

그림으로 살펴보면

![image](https://user-images.githubusercontent.com/60773356/134628743-12e40fdb-5744-4561-a733-a1d21258ed9d.png)

웹 서버를 통해 `Reverse Proxy`로 로드 밸런싱을 하고 있고 백엔드 서버 두개가 존재한다고 가정해보자. 또한 로드 밸런싱은 `Round Robin` 방식을 사용하고 있다고 가정하겠다.

![image](https://user-images.githubusercontent.com/60773356/134629290-55d98aaf-62e6-4cfc-88ec-b69116c2d198.png)

기본적으로 각 `WAS` 하나당 하나의 세션 저장소가 생성될 것이다.

`사용자 1`의 로그인 요청이 로드 밸런서에 의해 `Web 1`로 전달되었고 로그인이 성공했다면 `session 1`에 해당 사용자의 세션 정보가 저장된다.

다음 요청시에 이 세션 정보를 사용해 다시 로그인하지 않고 애플리케이션의 서비스를 이용할 수 있게 된다. 이것이 우리가 흔히 사용하는 로그인이다.

하지만 이것은 `사용자 1`의 요청이 항상 `Web 1`로만 전달되었을 때의 이야기이다.

로드 밸런싱의 전략으로 `Round Robin`을 사용하고 있기에 해당 사용자의 요청이 항상 `Web 1`로만 전달된다고 보장할 수 없다.

그럼 만약 `사용자 1`의 다음 요청이 `Web 2`로 전달된다면 어떻게 될까?

당연한 이야기이지만 `Web 2`는 `사용자 1`을 모르기 때문에(`session 2`에 `사용자 1`의 세션 정보가 없다!!) 다시 로그인을 요구할 것이다.

실제 서비스에서 이런 일은 절대 없어야한다. **이를 해결하기 위한 것이 바로 `Session Clustering`이다!!**

![image](https://user-images.githubusercontent.com/60773356/134630252-2a28ec77-e0ee-4a57-aafd-5ac528ed0ee4.png)

세션 클러스터링을 사용하게되면 세션 저장소를 공유하기 때문에 어떤 서버로 요청이 들어오던지 간에 한번 로그인한 사용자를 서버가 인지할 수 있다.


## Spring Boot와 Redis를 사용한 세션 클러스터링

`Spring Boot`와 `Redis`를 사용한다면 간단히 의존성 추가와 스프링 부트 설정만으로 세션 클러스터링을 손쉽게 사용할 수 있다.

`Spring`을 사용한다면 `@EnableRedisHttpSession`과 별도의 스프링 빈을 설정해줘야하는 등의 몇가지 설정을 만져줘야하지만,
`Spring Boot`와 함께 사용한다면 아래와 같이 한줄로 끝낼 수 있다.

```
spring:
  session:
    store-type: redis
```

나머지 `port`나 세션의 `timeout`설정은 기본값(6379포트, 30분)으로 사용하겠다.

`Spring Security`를 사용하고 있는 애플리케이션에 로그인을 하면 `Spring Session`이 자동으로 `Redis`에 저장된다.

`redis cli`에서 세션 값들을 살펴보자.

![image](https://user-images.githubusercontent.com/60773356/134632163-32c6a7ab-cea6-40ea-9318-c138b4140035.png)

현재 세션 저장소에는 아무것도 저장되어있지 않은 상태이다.

이제 애플리케이션에 회원가입을 하고 로그인을 해보자.

![image](https://user-images.githubusercontent.com/60773356/134632313-aa54bbb5-5472-46c1-b7cf-d31b001e0cc8.png)

세션 저장소에 위와 같이 세션 정보가 저장되었고 여기서 핵심은 `spring:session:sessions`이다.

`spring:session:sessions`의 필드 값을 살펴보면

![image](https://user-images.githubusercontent.com/60773356/134632845-d3e70d85-113f-454d-a093-dfc7d83e869f.png)

위와 같이 4개의 필드가 존재한다.
* 마지막 접속 시간
* 스프링 시큐리티 컨텍스트
* 세션 timeout 시간
* 세션이 만들어진 시간

![image](https://user-images.githubusercontent.com/60773356/134633234-906fd610-2a5f-4240-bfac-852125bdccc8.png)

각 필드에 이렇게 hash값이 저장되어있다.

이제 각 서버는 이 세션 저장소를 공유하고 이를 통해 클라이언트의 요청을 처리할 수 있게 된 것이다. 또한 세션이 만료되면 자동으로 `Redis`에서 지워지므로 매우 편리하게 사용이 가능하다.






