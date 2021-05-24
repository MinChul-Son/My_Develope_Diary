# 변경감지와 병합(merge)

## 준영속 엔티티
* 영속성 컨텍스트가 더는 관리하는 않는 엔티티를 의미한다.
* DB에 한번 저장되어 식별자가 존재하는 엔티티.
* persist() 메서드를 통해 영속성 컨텍스트에 담겼을 때는 식별자가 존재되고, 객체만 생성되었을 때는 식별자가 존재하지 않는다. 
* 따라서 식별자가 존재한다면 준영속 엔티티로 볼 수 있다.
* 준영속 엔티티를 수정하는 2가지 방법
  - **변경 감지 기능** 사용 ==> Dirty Checking
  - **병합** 사용 ==> Merge

----------------------------------------------
## 1. 변경 감지
![image](https://user-images.githubusercontent.com/60773356/119348307-a8263980-bcd7-11eb-89f9-b6ba37469ef6.png)
* Id를 기반으로 영속성 컨텍스트안의 영속성 엔티티를 가져온다.
* 이것을 파라미터로 넘어온 정보로 수정한다.(실제는 setter 사용 x, 별도의 메서드 사용해야함)
* 현재 findItem은 영속성 엔티티이므로, 따로 저장해주지 않아도 변경 감지 기능을 통해 JPA가 UPDATE문을 날려준다.
* @Transactional을 통해 DB에 commit이 되고, JPA가 flush()를 통해 변경을 감지하여 UPDATE


## 2. 병합
```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
	Item mergeItem = em.merge(item);
}
```
## merge 동작 과정
![image](https://user-images.githubusercontent.com/60773356/119348439-d4da5100-bcd7-11eb-87f5-7a2e78a2ff31.png)
* merge()를 실행
* 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회
* 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고, 1차 캐시에 저장
* 조회한 영속 엔티티에 준영속 엔티티의 값을 채움
* 영속 상태인 엔티티를 반환
* 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 UPDATE SQL이 실행


#### 주의: 변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면 모든 속성이 변경된다. 병합시 값이 없으면 null 로 업데이트 할 위험도 있다. 
----------------------------------------
### 엔티티를 변경할 때는 항상 변경 감지를 사용하자!!
* 트랜잭션이 있는 서비스 계층에 식별자( id )와 변경할 데이터를 명확하게 전달(파라미터 or dto)
* 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경
* 트랜잭션 커밋 시점에 변경 감지가 실행
* ex) 파라미터
  ```java
  @Transactional
    public void updateItem(Long itemId, String name, int price, int stockQuantity) {
        Item findItem = itemRepository.findOne(itemId);
        findItem.setName(name);
        findItem.setPrice(price);
        findItem.setStockQuantity(stockQuantity);
    }
  ```
* ex) DTO
  ```java
  @Transactional
    public void updateItem(Long itemId, UpdateDTO dto) {
        Item findItem = itemRepository.findOne(itemId);
        findItem.setName(dto.getName());
        findItem.setPrice(dto.getPrice());
        findItem.setStockQuantity(dto.getStockQuantity());
    }
  ```



