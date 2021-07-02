# 상속관계매핑
* 각각의 테이블로 변환
* 통합 테이블로 변환
* 서브타입 테이블로 변환

## 각각의 테이블로 변환(조인 전략)
* 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 PK를 받아 PK+FK를 PK로 사용한다.
* 테이블 타입을 구한하는 컬럼이 추가되어야함.
  - ex) 영화 테이블, 앨범 테이블, 도서 테이블을 구분할 수 있는
* ![image](https://user-images.githubusercontent.com/60773356/124255266-ef200e00-db64-11eb-9a80-256c502d9ccf.png)
```java
// 부모 엔티티
@Entity
@Inheritance(strategy = Inheritance.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
  @Id @GeneratedValue
  @Column(name = "ITEM_ID")
  private Long id;
  private String name;
  private int price;
}

// 자식 엔티티
@Entity
@DiscriminatorValue(name = "M")
public class Movie extends Item {
  private String director;
  private String actor;
}
```
* 상속 매핑은 부모 클래스에 @Inheritance 애노테이션을 사용해야함. 조인전략을 사용하므로 strategy = Inheritance.JOINED
* @DiscriminatorColumn는 부모 클래스에 구분 칼럼을 지정한다. 
* @DiscriminatorValue는 저장될 때 구분 칼럼에 입력될 value를 지정한다. 위의 코드에서 영화 엔티티를 저장하면 DTYPE의 값으로 "M"이 들어간다.
* Default로 부모의 ID 칼럼명(현재는 "ITEM_ID")를 그대로 사용하는데 @PrimaryKeyJoinColumn을 사용하여 변경할 수 있다.

### 장점
* 테이블 정규화
* FK 참조 무결성 제약조건을 활용할 수 있음
  - 외래키는 참조할 수 없는 값을 가질 수 없다는 규칙
* 저장공간을 효율적으로 사용

### 단점
* 조회할 때 조인이 많이 사용되어 성능 저하의 가능성 존재
* 조회 쿼리가 복잡
* 데이터를 등록할 때 INSERT 쿼리 두번 실행됨

-------------------------------------------------------

## 단일 테이블 전략(SINGLE TABLE)
* 테이블 하나에 값을 모두 저장하고 구분 칼럼으로 구분하는 전략이다.
* 조회할 때 조인이 사용되지 않기 때문에 일반적으로 가장 빠름.
* 이 때 자식 엔티티의 칼럼은 모두 null을 허용해야한다.
* ![image](https://user-images.githubusercontent.com/60773356/124257390-4aeb9680-db67-11eb-8afd-b9e7c4396961.png)
  - 영화를 저장하면 영화와 관련되지 않은 칼럼인 Artist, ISBN, Author에는 null이 들어감
```java
// 부모 엔티티
@Entity
@Inheritance(strategy = Inheritance.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
  @Id @GeneratedValue
  @Column(name = "ITEM_ID")
  private Long id;
  private String name;
  private int price;
}

// 자식 엔티티
@Entity
@DiscriminatorValue(name = "M")
public class Movie extends Item {...}
@Entity
@DiscriminatorValue(name = "A")
public class Album extends Item {...}
@Entity
@DiscriminatorValue(name = "B")
public class Book extends Item {...}
```

### 장점
* 조인이 없으므로 조회 성능 좋음
* 조회 쿼리 단순

### 단점
* 자식 엔티티의 컬럼은 모두 null을 허용해야함
* 테이블이 커질 수 있음 ==> 상황에 따라 조회 성능의 저하될 가능성 존재


-----------------------------------------------
## 구현 클래스마다 테이블 전략
* ![image](https://user-images.githubusercontent.com/60773356/124257964-df55f900-db67-11eb-9962-669bbe096217.png)
```java
// 부모 엔티티
@Entity
@Inheritance(strategy = Inheritance.TABLE_PER_CLASS)
public abstract class Item {
  @Id @GeneratedValue
  @Column(name = "ITEM_ID")
  private Long id;
  private String name;
  private int price;
}

// 자식 엔티티
@Entity
public class Movie extends Item {...}
@Entity
public class Album extends Item {...}
@Entity
public class Book extends Item {...}
```
* 자식 엔티티마다 테이블을 만드는 전략

### 장점
* 서브 타입을 구분해서 처리할 때 효과적
* not null 제약조건 사용 가능

### 단점
* 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION 사용)
* 자식 테이블을 통합해서 쿼리를 작성하기 어려움


> > > [자바 ORM 표준 JPA 프로그래밍] 도서를 보고 작성했습니다.

























