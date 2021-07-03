# 지연로딩과 프록시

## 설명을 위한 예시
* ![image](https://user-images.githubusercontent.com/60773356/124354227-f834dc00-dc45-11eb-9e53-7f71d40daf55.png)
* 다대일로 매핑되어 있는 회원과 팀의 관계를 생각해보자.
* 회원은 하나의 팀에 소속되며, 하나의 팀에는 여러 개의 회원이 소속될 수 있다.
-----------------------------------------------------

## 지연로딩? 즉시로딩?
* JPA는 연관된 엔티티의 조회 시점을 선택하는 두가지 방법을 제공하는데 그것이 바로 지연로딩과 즉시로딩이다.
* 즉시로딩은 회원 엔티티를 조회할 때 이와 연관된 팀 엔티티 또한 함께 조회한다.
```java
@ManyToOne(fetch = FetchType.EAGER)
```
* 지연로딩은 회원 엔티티를 조회할 때 팀 엔티티에는 Proxy라는 가짜 객체를 넣어두고 실제 팀 엔티티가 사용되는 시점에 조회를 하는 방법이다.
```java
@ManyToOne(fetch = FetchType.LAZY)
```

### 즉시로딩
* 가능하면 조인 쿼리를 사용하여 쿼리 한번으로 두 엔티티를 조회한다.
* member.getTeam()이 호출되면 로딩된 팀 엔티티를 반환해준다.
* 기본적으로 **null값이 허용되면 JPA는 즉시로딩의 조인전략으로 left outer join을 사용하고, 그렇지 않다면 inner join을 사용한다.**
* 외부 조인보다 내부 조인이 성능 최적화에 더 유리함.
* 이를 위해 연관 엔티티에 nullable=false 옵션을 추가해주면 내부 조인을 사용한다.
```java
@Entity
public class Member {
  @ManyToOne(fetch = FetchType.EAGER)
  @JoinColumn(name = "TEAM_ID", nullable = false) // 내부 조인 사용(null 값 허용 x)
  private Team team;
}
```
* **Collection을 2개 이상 즉시 로딩하는 것은 권장 X**
  - 성능 문제가 발생할 가능성이 높다.
* **Collection 즉시 로딩은 항상 외부 조인을 사용**

### 지연로딩
* 지연로딩을 사용하게 되면 em.find로 Member엔티티를 조회하면 Member엔티티만 조회하고 Team은 조회하지 않는다.
* 대신에 team 멤버 변수에 Proxy 객체를 넣어둔다.
* ![image](https://user-images.githubusercontent.com/60773356/124354623-d89eb300-dc47-11eb-8c37-e95ab1ea44cf.png)
* 이후 **getTeam()과 같이 팀에 대한 조회가 발생하면 그 때 팀 엔티티를 실제로 조회하고 반환해준다.(Proxy 초기화)**
* Proxy 객체는 실제 사용될 때까지 로딩을 미루기 때문에 지연로딩이라고 부른다.

-----------------------------------------------------------
## 프록시 
* 지연 로딩 기능을 사용하기 위해 실제 엔티티 객체 대신 가짜 객체를 집어 넣어 DB 조회를 지연시키는데 이를 **프록시 객체**라고 한다.

### em.find Vs em.getReference
* JPA에서 EntityManger.find()를 사용하면 영속성 컨텍스트에서 먼저 찾고 없으면 DB를 조회한다.
  - 실제 사용하든 안하든 DB를 조회하게 된다.
* 엔티티를 실제 사용하는 시점까지 DB 조회를 미루고 싶다면 EntityManager.getReference()를 사용한다.
  - 이 메서드를 사용하면 JPA는 DB를 조회하지 않고 실제 엔티티 객체 대신 DB 접근을 위임받은 Proxy 객체를 반환한다.

### Proxy 객체
* Proxy 클래스는 실제 클래스를 상속받아 만들어지므로 겉모양이 동일하다.
* Proxy 객체는 실제 객체에 대한 참조를 보관한다.
* 참조를 보관하기 때문에 Proxy 객체가 실제 객체의 메서드를 호출할 수 있는 것이다.
* ![image](https://user-images.githubusercontent.com/60773356/124354868-38e22480-dc49-11eb-9cc1-8b6085b1bb70.png)
* Proxy 객체는 해당 엔티티가 실제 사용될 때 DB를 조회하여 실제 엔티티 객체를 생성하는데 이를 **Proxy 객체의 초기화**라고 한다.
* ![image](https://user-images.githubusercontent.com/60773356/124354890-5f07c480-dc49-11eb-80c6-933f9525ab34.png)
* JPA가 제공하는 PersistenceUtils.isLoaded() 메서드를 사용하여 Proxy 객체의 초기화 여부를 확인할 수 있다.

### Proxy의 특징
* 사용할 때 한 번 초기화
* Proxy 객체가 초기화된다는 것은 Proxy 객체가 실제 엔티티가 된다는 의미가 아님. Proxy 객체를 통해 실제 엔티티에 접근할 수 있다는 의미
* 타입 체크에 주의
* 만약 영속성 컨텍스트에 찾는 엔티티가 있으면 DB를 조회할 필요가 없으므로 getReference()를 사용해도 실제 엔티티를 반환
* 준영속 상태의 Proxy를 초기화하면 오류 발생


### Collection Wraper
* Hibernate는 엔티티를 영속 상태로 만들 때 Collection이 있으면 이를 추적, 관리할 목적으로 원본 Collection을 Hibernate가 제공하는 내장 Collection으로 변환하는데 이를 Collection Wrapper라고 한다.
* 엔티티를 지연 로딩하면 Proxy 객체를 사용하지만, Collection은 Collection Wrapper가 이를 처리해준다.
* 이는 member.getOrders()와 같이 Collection 자체를 조회한다고 초기화되지 않는다.
* member.getOrders().get(0)와 같이 실제 내부 데이터를 조회할 때 DB를 조회하여 초기화한다.

-------------------------------------------------

#### 모든 연관관계에 지연로딩을 사용하도록하고 꼭 필요한 곳에서만 즉시로딩을 사용하자!!
#### 즉시 로딩은 성능 저하와 자주 연결되기 때문에 조심해서 사용해야한다.
#### 왜 즉시 로딩을 써야하고 성능 저하가 발생하지 않는지를 생각해봐야한다!






>>> 자바 ORM 표준 JPA 프로그래밍을 읽고 공부하였습니다.














