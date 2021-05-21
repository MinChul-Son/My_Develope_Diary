# 엔티티 매니저

![image](https://user-images.githubusercontent.com/60773356/119091119-8ca4ff80-ba47-11eb-998a-4f8181dc690e.png)
* JPA를 사용하게 되면 애플리케이션과 데이터베이스 사이에 영속성 컨텍스트(Persistence Context)를 두고 데이터를 관리한다.
* 영속성 컨텍스트를 사용하기 때문에 조회시 항상 동일한 객체 인스턴스를 반환함을 보장한다.
* 이를 관리하는 것이 바로 **Entity Manager**이다.
* @PersistenceContext 애노테이션을 사용해 Entity Manager를 주입받아 사용하면 된다.
* [공식문서](https://docs.jboss.org/hibernate/entitymanager/3.6/reference/en/html_single/#d0e46)

-------------------------------------------
### 참고로, 스프링 데이터 JPA에서는 @Autowired를 사용해 Entity Manager를 주입받을 수 있게 지원해준다!!!!
* 코드 예시
```java
@Repository
@RequiredArgsConstructor
public void SampleRepository {
  
  private final EntityManager em;
  
  public void save(Sameple sample){
    em.persist(sample);
  }
}
```

**완전 편함!!!🤔**
