# 데이터베이스 락(Lock)

이 글은 우아한테크코스의 `10분 테코톡`을 보고 작성된 글입니다.

[해당 영상](https://www.youtube.com/watch?v=w6sFR3ZM64c&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)

## 정의

Recored locking is the technique of preventing simultaneous access to data in a database, to prevent inconsistent results.

DB의 **일관성**과 **무결성**을 유지하기 위해 트랜잭션의 순차적 진행을 보장할 수 있는 직렬화 장치

![image](https://user-images.githubusercontent.com/60773356/135597116-dcccaf11-f5b0-4fa0-abe6-29234660a2f8.png)
* 위의 경우에 **동시성 문제**로 계좌의 잔액이 0 이하로 떨어지는 `side effect`를 방지하기 위한 장치(마이너스 통장 아님)

위와 같은 문제를 방지하고 데이터의 `일관성`을 지키기위해 `LOCK~을 걸고 관리하는 것을 `Locking`이라고 한다. 


## 필요성
`동시성`

동시에 접근하려 수정하려할 때 DB의 일관성이 깨질 수 있음.

#### 송금 시스템, 도서 관리 시스템, 수강 신청 시스템, 멀티 트랜잭션 환경에서 필요
ex)
- 계좌 잔액이 1000원이고 A에게 2000원, B에게 3000원을 입금받았는데, 내 계좌에는 4000원만 남아있음 ==> 동시성 문제, lock이 필요
- `이펙티브 자바`는 한권인데 두명이 동시에 빌린 것으로 되어있음 ==> 동시성 문제, lock이 필요


## 종류와 사용법
### Optimistic Lock(낙관적인)
- 낙관적인 : 기본적으로 데이터 갱신시 충돌(동시성 문제)이 발생하지 않을 것이라고 낙관적으로 보는 것
- 비선점적인 : 낙관적으로 예상하기 때문에 우선적으로 `lock`을 걸지 않음

그림을 통해 살펴보자.

![image](https://user-images.githubusercontent.com/60773356/135598210-a7b69e6d-acd2-464f-bb5e-1296355666ac.png)

`Optimistic Lock`은 버전을 사용하여 관리를 한다.

그림과 같이 `Client1`이 `Order`에 접근을 할 때 `Version 1`의 `Order`에 접근한다. 마찬가지로 `Client2`도 `Version 1`의 `Order`에 접근한다.

`Client1`이 `Order`에 어떤 것을 추가하고 `Commit`을 자신이 접근한 `Version 1`에 +1을 하여 `Version 2`로 날리게된다. 이제 `DB`의 `Order` 버전은 '2'이다.

`Client2`도 `Order`에 어떤 것을 추가하고 마찬가지로 `Version 2`로 `Commit`을 날리는데, 이미 `Order`의 버전은 `2`이기 때문에 `OptimisticLockException`을 발생시킨다.

`JPA`를 사용할 때의 예시 코드이다.
```java
@Entity
public class OptimisticLockingStudent {
  
  @Id
  private Long id;
  
  private String name;
  private String lastName;
  
  @Version
  private Integer version;
}
```

위와 같이 `@Version`을 사용한 필드를 두게되면 자동으로 `commit`시에 버전이 올라가고 충돌을 방지해준다.

**데드락 가능성이 적고 성능의 이점을 가짐**

**하지만 충돌이 발생하면 오버헤드가 발생**

### Pessimistic Lock(비관적인)
- 비관적인 : 기본적으로 데이터 갱신시 충돌이 발생할 것이라고 비관적으로 보고 `lock`을 설정
- 선점적인 : 비관적으로 보기 때문에, 우선적으로(조회할 때부터) `lock`을 검

![image](https://user-images.githubusercontent.com/60773356/135599685-ce3c33b3-c15e-4fea-ae2a-b593a00cee20.png)

조회를 할 때부터 `lock`을 걸어 다른 곳에서 접근할 시에 `wait time`이 발생 

여기에는 두가지 동류가 존재하는데 `Shared Lock`과 `Exclusive Lock`이 존재한다.
- Shared Lock : 다른 사용자가 동시에 읽을 수는 있지만, 수정/삭제를 방지
- Exclusive Lock : 다른 사용자의 읽기/수정/삭제를 모두 방지

```java
@Entity
public class PessimisticLockingStudent {
  
  @Id
  private Long id;
  
  private String name;
}

#----------------------------------

PessimisticLockingStudent student = entityManager.find(PessimisticLockingStudent.class, 1L);
entityManager.refresh(student, LockModeType.PESSIMISTIC_WRITE); // Exclusive Lock

PessimisticLockingStudent student2 = entityManager2.find(PessimisticLockingStudent.class, 1L);
entityManager2.refresh(student2, LockModeType.PESSIMISTIC_WRITE); // 예외 발생!!!!

```

**충돌에 대한 오버헤드가 줄어들고, 무결성을 지키기 용이**

**충돌이 없으면 오버헤드가 발생**


## 적용
어떤 상황인지를 판단하는 것이 중요함.(Lock은 비용이 비쌈)
- 충돌이 자주 발생하는 상황인가?
- 읽기와 수정의 비율이 어디에 가까운가?

일반적으로 웹 애플리케이션은 `Optimistic Lock`을 주로 사용한다.
