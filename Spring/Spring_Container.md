# Spring Container😛

## ApplicationContext 를 스프링 컨테이너라 한다.
* 정확히는 스프링 컨테이너를 부를 때 BeanFactory , ApplicationContext 로 구분해서 이야기 한다. 하지만 BeanFactory 를 직접 사용하는 경우는 거의 없으므로 일반적으로 ApplicationContext 를 스프링 컨테이너라 한다.
* 스프링 컨테이너는 @Configuration 이 붙은 AppConfig 를 설정(구성) 정보로 사용한다. 
* 여기서 @Bean 이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 
  - 스프링 컨테이너에 등록된 객체를 스프링 빈이라 한다.
  - 스프링 빈은 @Bean 이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다.
  - @Bean (name ="바꿀이름") 을 사용하여 이름을 바꿀 수 도 있음.
* 스프링 컨테이너를 통해서 필요한 스프링 빈(객체)를 찾아야 한다. 스프링 빈은 applicationContext.getBean() 메서드 를 사용해서 찾을 수 있다.
* ApplicationContext는 **인터페이스**이다.
----------------------
### Spring Container 생성과정
#### 1. Spring Container 생성
![image](https://user-images.githubusercontent.com/60773356/120153224-6b9f9400-c229-11eb-8a90-e869e8d7c29e.png)
```
new AnnotationConfigApplicationContext(AppConfig.class)
```
* 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 한다.

#### 2. Spring Bean 등록
![image](https://user-images.githubusercontent.com/60773356/120153366-9ab60580-c229-11eb-824a-7df838f5b49d.png)
* 스프링 컨테이너는 parameter로 넘어온 설정 클래스(AppConfig) 정보를 사용해서 스프링 빈을 등록
* @Bean 애노테이션 붙은 것들을 모두 호출하여 key-value의 형태로 등록시킴
* 빈 이름은 메서드 이름을 사용하는데, 직접 부여할 수도 있음.
* 빈 이름은 항상 다른이름을 부여해야함.
  - 같은 이름을 부여하면, 다른 빈이 무시되거나, 기존 빈을 덮어버리거나 설정에 따라 오류가 발생
* 최근에는 spring boot에서 같은 이름의 빈이 들어오면 튕겨냄(default설정으로 되어 있음)

#### 3. 스프링 빈 의존관계 설정 - 준비
![image](https://user-images.githubusercontent.com/60773356/120153457-b4efe380-c229-11eb-95ea-5a2ea374fd77.png)

#### 4. 스프링 빈 의존관계 설정 - 완료
![image](https://user-images.githubusercontent.com/60773356/120153484-bde0b500-c229-11eb-853b-b3989d2ab3ec.png)
* 스프링 컨테이너는 설정 정보를 참고해서 의존 관계를 주입(DI)한다.
* 동적인 객체 인스턴스 의존관계를 spring이 연결해줌.(참조값들이 연결되는 것)
* 스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져 있다.
* 스프링 객체를 생성하고, 그 후에 서로를 엮어준다.
* 하지만 위와 같이 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리된다.



------------------------------
-----------------------------
## BeanFactory와 ApplicationContext
![image](https://user-images.githubusercontent.com/60773356/120153720-08623180-c22a-11eb-95ac-21b8653abd9b.png)

### BeanFactory
* 스프링 컨테이너의 최상위 인터페이스
* 스프링 빈을 관리하고 조회하는 역할 담당
* getBean()을 제공
* Bean을 조회하고 꺼내는 등의 대부분 기능은 BeanFactory가 제공하는 기능

### ApplicationContext
* BeanFactory기능을 모두 상속받아 제공한다.
* 애플리케이션을 개발할 때 빈은 관리 / 조회 기능과 더불어 수 많은 부가 기능이 필요
* 그 부가 기능을 제공

#### ApplicationContext가 제공하는 부가기능
![image](https://user-images.githubusercontent.com/60773356/120153898-39426680-c22a-11eb-8834-134400dd0155.png)
* MessageSource : 메시지소스를 활용한 국제화 기능
  - ex) 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
* EnvironmentCapable : 환경변수
  - 로컬, 개발, 운영등을 구분해서 처리
    + 로컬 개발환경 : 현재 내 PC에서 개발하는 환경
    + 개발환경(테스트서버) : 테스트서버에 올려서 여러 시스템을 엮어 서버에 띄워두고 테스트할 수 있는 환경
    + 운영환경 : 실제 production에 나가게되는 환경
  - 각 환경별로 어떤 DB에 연결해야될지...같은 설정을 해줌
* ApplicationEventPublisher : 애플리케이션 이벤트
  - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
* ResourceLoader : 편리한 리소스 조회
  - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회
  - 추상화하여 내부에서 편리하게 쓸 수 있는 기능을 제공



>  > 김영한님의 강의를 듣고 공부하였습니다.






