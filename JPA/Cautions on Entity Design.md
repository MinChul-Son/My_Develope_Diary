# 엔티티 설계 시에 주의할 점!!

## 엔티티에는 가급적 Setter를 사용하지 말자!
* Setter가 모두 열려있으면 변경 포인트가 많아지고, 어디서 변경되었는지를 알아내기가 어려워진다.
  - 유지보수가 어렵다.


## 모든 연관관계는 지연로딩으로 설정(FetchType.LAZY)
* EAGER은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. JPQL을 실행할 때 N + 1 문제가 자주 발생
  - EAGER는 하나의 객체를 DB로부터 읽어올 때 참조 객체들의 데이터까지 전부 읽어오는 방식을 뜻한다.
* 모든 연관관계는 LAZY(지연로딩)으로 설정해야함.
  - 반대로 LAZY 타입은 게을러서, 참조 객체들의 데이터들은 무시하고 해당 엔티티의 데이터만을 가져온다.
* 연관된 엔티티를 함께 DB에서 조회해야 하면, fetch join 또는 엔티티 그래프 기능을 사용
* @OneToOne, @ManyToOne 관계는 default가 EAGER이므로 LAZY로 설정해야함.
![image](https://user-images.githubusercontent.com/60773356/119071906-9453ac80-ba25-11eb-8918-2427123d1ba3.png)
* @OneToMany는 default가 LAZY
![image](https://user-images.githubusercontent.com/60773356/119071930-9e75ab00-ba25-11eb-90a3-5f20cd69e1f5.png)


## 컬렉션은 필드에서 초기화!!
* null 문제에서 안전함.

* Hibernate는 엔티티를 영속화(persist)할 때, 컬랙션을 감싸서 Hibernate가 제공하는 내장 컬렉션으로 변경.
* 만약 임의의 메서드에서 컬렉션을 잘못 생성하면 Hibernate 내부 메커니즘에 문제가 발생할 수 있음.

* 필드 레벨에서 생성하는 것이 가장 안전하고, 코드도 간결!
