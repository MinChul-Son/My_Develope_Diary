# Redis 🤩
이 글은 우아한테크코스의 `10분 테코톡`을 보고 작성된 글입니다.

[해당 영상](https://www.youtube.com/watch?v=Gimv7hroM8A&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)

## Redis 개요
#### Remote dictionary server, DB, Cache, Message broker, In-memory Data Structure Store, Supports rich data structure

`Redis`란 `Remote dictionary server`의 줄임말로써, 외부에 있는 dictionary(자바의 HashMap과 유사한 Key-Value의 자료구조) 서버이다.

`Redis`는 `In-memory Data Structure Store`로써 메모리 상에 데이터를 저장한다. 즉, **휘발성**이다.


## Cache 개념
`Cache`는 나중의 요청에 대한 결과를 미리 저장했다가 빠르게 사용하는 것을 말한다.

아래의 메모리 계층도를 살펴보자.

![image](https://user-images.githubusercontent.com/60773356/135717676-7b31207b-331a-48ab-a02a-e35189ae7818.png)
* 위로 갈수록 빠르고 비용이 높고, 아래로 갈수록 느리고 비용이 낮다.

대부분의 `DB`는 `HDD`, `SDD`에 데이터를 저장하지만 이는 비교적 느리기 때문에 `Main Memory`에 더 빠르고 쉽게 데이터에 접근하기위해 나온 것이 `Redis`이다.
(하드웨어의 발달로 인해 `Main Memory`의 크기도 적당히 커졌기 때문에)


## Redis 자료구조
1. String

![image](https://user-images.githubusercontent.com/60773356/135718035-052731ab-6a8a-4f6e-9aff-acacb5bfc677.png)

2. List

![image](https://user-images.githubusercontent.com/60773356/135718055-2926e1b6-6c41-4dac-9749-9149b77a45c1.png)

3. Set

![image](https://user-images.githubusercontent.com/60773356/135718067-e37aee60-5766-4098-9f38-ec4894cb3511.png)

4. Sorted Set

![image](https://user-images.githubusercontent.com/60773356/135718081-8d9f2937-0076-4569-a604-e46c09601208.png)
* Score라는 숫자로 순서를 정렬

5. Hash
![image](https://user-images.githubusercontent.com/60773356/135718099-ad1892f2-11a9-4289-a156-9d0a7b9e7fbe.png)


#### 이것들은 `Java`의 자료구조와 어떤 차이점을 가지고 있을까??

`Java`의 `HashMap`을 사용하게 되면 동일하게 `In-memory DB` 아님?
==> 서버가 여러대인 경우 동시성 문제가 발생(ex: 세션 문제)
==> 멀티 쓰레드 환경에서 경쟁 조건이 발생할 가능성

`Redis`는 기본적으로 `Single Thread`이다. 또한 `Redis`의 자료구조는 `Atomic`(원자적)인 성질을 가지고 있어 `Critical Section`(임계구역)에 대한 동기화를 제공한다.



### 어디서 쓰면 좋나요?
여러 서버에서 같은 데이터를 공유할 때 ==> `Redis`를 통한 세션 클러스터링!!

`Atomic`한 자료구조 또는 `Cache`를 사용하기 위해!!

## Redis 주의사항

##### `Redis`는 앞서 언급했듯이 `Single Thread` 서버이기 때문에 시간 복잡도를 고려해야한다.
![image](https://user-images.githubusercontent.com/60773356/135718442-edb97896-5347-45cc-867b-5bf54505397c.png)
* 네트워크로부터 명령을 받아서 요청을 처리하는데 `Command`가 오랜시간이 걸리는 경우 나머지 요청들이 더 이상 받아지지않고 서버가 다운되는 문제가 발생할 수 있음
* 따라서 `O(N)`을 가지는 명령어는 주의하여 사용해야함 ==> `KEYS *`, `FlushAll`, `GetAll`등 모든 데이터를 가져오는 `O(N)`의 명령어들


#### `In-memory` 특성상 메모리 파편화, 가상 메모리등의 이해가 필요하다.
**메모리 파편화**
![image](https://user-images.githubusercontent.com/60773356/135718539-c35d6367-8b3e-4176-9e64-3c78af622f33.png)
* 메모리를 해제하는 과정에서 그림과 같이 사이사이 빈 공간들이 생김
* 새로운 큰 데이터를 저장하려할때 빈 공간이 존재하지만 들어갈 수 없기 때문에 실제 사용하는 것보다 더 큰 메모리를 사용하고 있는 것으로 컴퓨터가 인식
* 이 과정에서 프로세스가 죽는 현상이 발생할 수 있음

**가상메모리 Swap**
![image](https://user-images.githubusercontent.com/60773356/135718610-786bf974-0bea-4711-bbff-e3fe9e582f94.png)
* 메인 메모리에 프로세스를 올릴 때 모두 올리는 것이 아니라 필요한 부분 부분 올려서 프로세스를 실행함
* 이러한 `Swap`과정에서 `latency`가 발생하고 길어지게 된다면 문제가 발생할 가능성이 있음

**Replication-Fork**
![image](https://user-images.githubusercontent.com/60773356/135718682-1b212b76-c080-4ae7-900c-ed568cc5825c.png)
* `Redis`는 휘발성이기 때문에 데이터가 유실될 가능성이 있음
* 따라서 복사본을 다른 곳에 저장하는 방식을 사용
* 이 과정에서 프로세스를 동일하게 복제하는 `Fork` 방식을 사용함
* 만약 메모리가 가득차있다면 복사본이 제대로 생성되지않고 서버가 죽는 현상이 발생할 수 있음


#### 즉, 메모리를 여유있게 사용해야해요!!!!
