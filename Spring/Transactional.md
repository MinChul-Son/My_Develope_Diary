# @Transactional
## 스프링에서는 트랜잭션 처리를 지원하는데 그중 어노테이션 방식으로 @Transactional을 선언하여 사용하는 방법이 일반적이며, 선언적 트랜잭션이라 부른다.
* 클래스, 메서드위에 @Transactional 이 추가되면, 이 클래스에 트랜잭션 기능이 적용된 프록시 객체가 생성된다.
* 이 프록시 객체는 @Transactional이 포함된 메소드가 호출 될 경우, PlatformTransactionManager를 사용하여 트랜잭션을 시작하고, 정상 여부에 따라 Commit 또는 Rollback 한다. 오류가 발생했을 시 자동으로 Rollback을 실행한다.
* 만약 테스트코드에서 해당 애노테이션이 사용되면 테스트 수행후 Rollback을 실행시켜준다.
* 당연한 소리지만, 테스트 코드가 실제 DB에 반영되어선 안되기 때문이다.(반복적인 테스트도 불가능)
* 이는 @Rollback(false)를 통해 자동 Rollback을 해제하고 테스트가 실제 DB에 반영된 결과를 확인할 수도 있다.

### 추가로 readOnly 옵션을 통해 읽기 전용 모드로 트랜잭션을 수행하여 연산을 최적화할 수 있다.
* 만약 조회하는 읽기 메서드가 많고, 변경이나 저장하는 쓰기 메서드가 많이 없다면 아래와 같이 클래스 타입에 읽기 전용으로 트랜잭션을 선언하고 쓰기 메서드에서만 따로 지정해주는 것이 좋다.
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {

    private final ItemRepository itemRepository;

    @Transactional
    public void saveItem(Item item) {
        itemRepository.save(item);
    }

    public List<Item> findItems() {
        return itemRepository.findAll();
    }

    public Item findOne(Long itemId) {
        return itemRepository.findOne(itemId);
    }
}
```
