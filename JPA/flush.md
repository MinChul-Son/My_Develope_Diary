# 플러시(flush)
* 플러시는 영속성 컨텍스트의 변경 내용을 DB에 반영한다.
## 플러시 동작 과정
* Dirty Checking 동작 ==> 스냅샷과 비교
* 쓰기 지연 SQL 저장소에 있는 쿼리들 DB로 전송

## 플러시 호출 방법
1. em.flush() 직접 호출
2. 트랜잭션 커밋 시 자동 호출
  - @Transactional이 선언된 클래스, 메서드에서 트랜잭션 종료 시점에 자동으로 커밋이 되고 flush호출됨

3. JPQL 쿼리 실행시 자동 호출