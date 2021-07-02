# @MappedSuperclass
* 부모는 테이블과 매핑하지 않고 매핑 정보만 자식에게 제공하고 싶을 때 사용하는 방법이다.
* **Spring Data JPA의 Auditing 기능과 함께 사용할 때 굉장히 유용하다.**
  - 엔티티의 생성일, 수정일
* ![image](https://user-images.githubusercontent.com/60773356/124259091-1a0c6100-db69-11eb-8916-f69a34ae8620.png)
```java
// 부모
@MappedSuperclass
public abstract class BaseEntity {
  @Id @GeneratedValue
  private Long id;
  private String name;
}

// 자식
@Entity
public class Member extends BaseEntity {
  private String teamName;
}
```
* 물려받은 매핑 정보를 재정의하려면 @AttributeOverride나 @AttributeOverrides 사용한다.
* 연관관계를 재정의하려면 @AssociationOverride나 @AssociationOverrides를 사용한다.

### Auditing 기능과 함께 사용
#### 순수 JPA
![image](https://user-images.githubusercontent.com/60773356/124260228-699f5c80-db6a-11eb-85da-46fc9d7b1bd0.png)
![image](https://user-images.githubusercontent.com/60773356/124260247-6f953d80-db6a-11eb-9b35-1aef5b320586.png)
* 자바8에서 제공하는 LocalDateTime을 사용한다.
* @PrePersist와 @PreUpdate를 사용하여 등록일, 수정일을 관리한다.

#### Spring Data JPA
![image](https://user-images.githubusercontent.com/60773356/124260400-9e131880-db6a-11eb-8f79-675c357fae1e.png)
* 먼저 설정 클래스에 @EnableJpaAuditing 애노테이션을 추가
![image](https://user-images.githubusercontent.com/60773356/124260533-c13dc800-db6a-11eb-8694-bc2f4eb8501d.png)
* 등록자, 수정자는 추가로 AuditorAware를 스프링 빈으로 등록해주어야한다.
* 세션 정보나 스프링 시큐리티 로그인 정보를 사용하여 등록자, 수정자를 추가함

