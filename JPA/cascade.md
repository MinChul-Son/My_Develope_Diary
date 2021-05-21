# 연관 관계(OneToMany, OneToOne....)에서의 cascade 옵션

## cascade?
* 영속성 전이를 뜻한다.
* 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티들도 영속 상태로 만들어 줌.
* 옵션
  - ALL : 모두 적용
  - PERSIST : 영속
  - REMOVE : 삭제
  - MERGE : 병합
  - REFRESH : REFRESH
  - DETACH : DETACH 
* 예를 들어, 부모-자식 관계로 있는 부모, 자식1, 자식2가 있다고 가정해보자.
```java
private final EntityManager em;
em.persist(parent);
em.persist(child1);
em.persist(child2);
```
  - 이와 같이 각 엔티티마다 persist를 해줘야함.
* 연관관계 매핑 시에 cascade를 사용하게 되면 부모만을 persist함으로써 자식까지 자동으로 처리된다.
```java
@OneToMany(mappedBy = 'parent', cascade = CascadeType.ALL)
..................생략.......
private final EntityManager em;
em.persist(parent);
```




## Example) 엔티티
![image](https://user-images.githubusercontent.com/60773356/119068583-7420ef00-ba1f-11eb-96be-872f62ba0de0.png)
* 위와 같은 엔티티 관계에서 Delivery와 OrderItem은 오직 Order에 의해서만 참조되고 order와 **LifeCycle**이 동일하기 때문에 cascade옵션 사용가능!!
* 하지만 만약 Delivery나 OrderItem이 중요한 로직이라 Order 뿐만 아니라 다른 곳에서도 참조가 일어난다면 cascade를 사용해선 안된다.
* 이 경우에는, 따로 Repository를 만들어 관리하고 처리해주어야한다.
