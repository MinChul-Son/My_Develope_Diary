# IoC(Inversion of Control) : 제어의 역전
* 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다.
* 한 마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다. 개발자 입장에서는 자연스러운 흐름이다.
  - 개발자가 직접 객체를 생성, 의존성을 맺고, 초기화 등등
* 하지만 스프링에서 스프링 컨테이너에 Bean(객체)를 등록만하면, 스프링 컨테이너에서 이를 모두 관리(생성, 의존성 설정, 초기화, 소멸)해준다.
* 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라 한다.
### 장점
* 코드의 재사용성과 유지보수성이 높아짐.

------------------------------------------------------------------------
# DI(Dependency Injection) : 의존관계 주입
## 정적인 클래스 의존관계
![image](https://user-images.githubusercontent.com/60773356/119222843-4c21b080-bb31-11eb-8bff-871242978282.png)
* 클래스가 사용하는 import 코드만 보고 의존관계를 쉽게 판단할 수 있다.
* 정적인 의존관계는 애플리케이션 을 실행하지 않아도 분석할 수 있다.
* 클래스 다이어그램을 보면 OrderServiceImpl 은 MemberRepository , DiscountPolicy 에 의존한다는 것을 알 수 있다.
* 그런데 이러한 클래스 의존관계 만으로는 실제 어떤 객체가 OrderServiceImpl 에 주입 될지 알 수 없다.
![image](https://user-images.githubusercontent.com/60773356/119222869-69567f00-bb31-11eb-99c2-5ea01cd903fe.png)
* 이런 코드들을 보고 Member, DiscountPolicy, MemberRepository와 의존관계에 있다는 사실을 알 수 있음
 
 ## 동적인 객체 인스턴스 의존 관계
 ![image](https://user-images.githubusercontent.com/60773356/119222889-7f643f80-bb31-11eb-987f-c897d403f937.png)
* 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존관계

![image](https://user-images.githubusercontent.com/60773356/119222897-88eda780-bb31-11eb-9aa2-90afc48cb620.png)
* 애플리케이션 실행 시점(런타임)에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것을 **의존관계 주입**이라 한다.
* 객체 인스턴스를 생성하고, 그 참조값을 전달해서 연결된다.
* 의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.
* 의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다.

----------------------------------------------------------------
### Spring이 이 역할을 해준다.
* DI 사용 X
  ```java
  public class Wheel {
    ....
    ....
  }
  ```
  
  ```java
  public class Car {
    private Wheel wheel;
    
    public Car(){
      wheel = new Wheel();
    }
  }
  ```
  - 이와 같이 의존성을 직접 주입해야한다.

* Spring Container를 사용하면?
  ```java
  @Component
  public class Wheel {
    ....
    ....
  }
  ```
  
  ```java
  public class Car {
    @Autowired
    private Wheel wheel;
  }
  ```

  - 스프링이 등록된 스프링 빈 중에 찾아서 자동으로 주입해준다.
 
  
  
  
  
  
  
