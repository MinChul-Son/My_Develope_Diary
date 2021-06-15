# @Configuration
![image](https://user-images.githubusercontent.com/60773356/122054348-8f1a2f80-ce22-11eb-8d36-bdc5a8f0529d.png)
* 위와 같은 Configuration 애노테이션을 사용하는 스프링 빈을 등록하는 스프링 설정 파일이 있다.
* 스프링 빈이 등록될 때 memberService()는 memberRepository()를 호출한다.
  - 이는 new MemoryMemberRepository를 호출하여 새로운 객체를 생성한다.
* orderService() 또한 memberRepository를 호출한다.
  - 이는 new MemoryMemberRepository를 호출하여 새로운 객체를 생성한다.

* 그렇다면 **싱글톤**이 깨지는 것이 아닌가??
--------------------------------
## 그렇지 않다!
* 자바 코드만을 보고 이야기 한다면 위에서 이야기한 바가 맞다.
* 하지만 스프링은 **클래스의 바이트코드를 조작하는 라이브러리**를 사용하여 싱글톤을 보장해준다!
![image](https://user-images.githubusercontent.com/60773356/122055227-5e86c580-ce23-11eb-8087-eb9aee7533e0.png)
* AppConfig가 스프링 빈으로 등록될 때 자동으로 CGLIB이라는 바이트코드 조작 라이브러리를 사용하여 AppConfig 클래스를 상속받는 임의의 다른 클래스를 만들어 스프링 빈으로 등록한다.
* 이 클래스가 싱글톤을 보장하도록 도와준다.
* Bean애노테이션이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 존재하지 않는다면 새로 생성하여 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.

### 이 덕분에 싱글톤을 보장할 수 있는 것이다.
### 스프링 설정 정보는 항상 @Configuration을 붙이자!
