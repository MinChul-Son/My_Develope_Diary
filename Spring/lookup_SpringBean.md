# Spring Bean 조회

## 기본적인 조회 방법
* getBean(빈이름, 타입)
* getBean(타입)
* 조회 대상 스프링 빈이 없으면 예외 발생
* NosuchBeanDefinitionException : No bean named "빈이름" available
* 스프링 컨테이너에 구현체가 등록만 되어있으면 조회가 됨
* 스프링 빈에 등록되어 있는 인스턴스 타입(MemberServiceImpl)을 보고 결정을 하기 때문에 꼭 인터페이스가 아니어도 된다.
* 하지만 **구체를 적는 것은 좋지 않음**
* 이것은 **역할(interface)에 의존하는 것이 아닌 구현에 의존하기 때문**
  - 필요에 따라 사용할 수 도 있음.


### 동일한 타입의 빈이 여러개 존재
![image](https://user-images.githubusercontent.com/60773356/120155169-85da7180-c22b-11eb-91f4-eb56879b739c.png)
* getBean(타입)으로 스프링 빈을 조회할 때 해당 타입의 스프링 빈이 여러개이면 위와 같은 오류가 발생한다.
* 이럴 땐 두가지 방법이 존재한다.
  - getBean(빈이름, 타입)을 통해 스프링 빈의 이름을 지정해주는 방법
  - getBeansOfType을 사용
    + 이 메소드의 return 타입은 Map으로 Key-Value형식으로 반환


### 상속 관계
![image](https://user-images.githubusercontent.com/60773356/120155491-d81b9280-c22b-11eb-9749-7e0b554a8319.png)
* 부모 타입으로 조회하면, 자식 타입도 함께 조회된다. 따라서 모든 자바 객체의 최고 부모인 Object타입으로 조회하면 모든 스프링 빈을 조회할 수 있다.
* 최상위 노드인 1번을 조회하면 자식인 2,3,4,5,6,7까지 모두 조회된다.
