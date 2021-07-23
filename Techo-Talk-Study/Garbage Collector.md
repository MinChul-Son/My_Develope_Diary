# JVM의 Garbage Collector
10분 테코톡 - 던님의 "JVM의 Garbage Collector"를 시청하고 작성한 글입니다.

[영상링크](https://www.youtube.com/watch?v=vZRmCbl871I&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)

`JVM`이란 `Java Virtual Machine`으로 운영체제의 메모리 영역에 접근하여 메모리 관리하는 프로그램이다. 메모리 관리, Garbage Collector 역할을 수행한다.

![image](https://user-images.githubusercontent.com/60773356/126794500-3d5cd72e-7eb0-473d-9f25-96fb1890da9f.png)

## Garbage Collector
**동적으로 할당한 메모리 영역** 중 사용하지 않는 영역을 탐지하여 해제하는 기능을 담당한다.

여기서 말하는 동적으로 할당한 메모리 영역은, `Heap`을 뜻한다. `Heap`에는 모든 `Object` 타입의 데이터가 할당되고 `Heap`영역의 `Object`를 가리키는 참조 변수가 `Stack`에 할당된다.

`Stack`은 정적으로 할당한 메모리 영역으로, `Stack`에는 원시 타입의 데이터가 값과 함께 할당되고 `Heap`영역에 생성된 `Object` 타입의 데이터의 참조 값이 할당된다.

 ```java
 public class Hello {
  public static void main(String[] args) {
      int num1 = 10;
      int num2 = 5;
      int sum = num1 + num2;
      String name = "던";
    }
 }
 ```
 ![image](https://user-images.githubusercontent.com/60773356/126795484-367b6df4-edc8-4b17-9959-ec1269f83823.png)
* 해당 클래스의 메인 메서드가 실행될 때 int와 같은 원시 타입의 데이터는 `Stack`영역에 그림과 같이 값과 함께 할당, 또한 `Heap`영역의 참조 변수도 할당
* String과 같은 Object 타입(객체) 데이터는 `Heap`영역에 그림과 같이 할당
* 메인 메서드가 종료되면 `Stack`영역은 모두 `pop`되어 사라지지만 `Heap`영역의 데이터(객체)들은 사라지지 않고 남아있게 된다.
* 존재하지만 접근할 수 없기 때문에 `Unreachable Object`라고 부르고 이것이 바로 `GC`의 대상이다.

## Garbage Collector 과정
1. `Garbage Collector`가 `Stack`의 모든 변수를 스캔하면서 각각 어떤 객체를 참조하고 있는지 찾아서 마킹
2. `Reachable Object`가 참조하고 있는 객체도 찾아서 마킹한다.
3. 마킹되지 않은 객체를 `Heap`에서 제거한다. ==> 마킹되지 않은 것이 바로 `Unreachable Object`

1,2번 과정이 마킹을 한다는 뜻에서 **Mark** 라고 하고, 3번 과정을 쓸어내린다라는 뜻에서 **Sweep** 이라고 한다. Garbage Collector의 과정을 **Mark & Sweep**이라고 한다.

### Garbage Collection은 언제 일어날까?
`Heap`의 구조는 아래와 같다.

![image](https://user-images.githubusercontent.com/60773356/126797237-48c8d4fd-6488-4135-9caa-b71e2a8ebbad.png)

`Heap`은 크게 `New Generation` 영역과 `Old Generation` 영역으로 나누어진다. 먼저 `New Generation`을 살펴보자.

`New Generation` 영역은 `Eden`, `Survival 0`, `Survial 1`으로 다시 나누어져있는데, 새로운 객체가 생성되면 `Eden` 영역에 할당된다.

계속 할당되다가 `Eden`영역의 메모리가 가득 차면 `GC`(Garbage Collection)가 발생한다. 이 때의 `GC`를 `Minor GC`라고 한다.

`Eden`영역에서 `GC`를 통해 `Mark&Sweep`이 발생하고 그 중에 살아남은 객체(Reachable Object)는 `Survival 0`영역으로 옮겨진다. 이 과정이 계속해서 반복된다.

반복이 되다가 `Survival 0` 또한 마찬가지로 가득 차게되면, `Survival 0` 영역에 대해서도 `Mark&Sweep` 과정이 수행된다.

`Survival 0`영역의 `GC`에서 살아남은 객체들은 `Survival 1`영역으로 이동하게 되고, **이동한 객체는 Age 값이 증가**한다.

이 때 중요한 것이 두 `Survival`영역 중 하나는 반드시 아무것도 없는 빈 상태여야 한다는 것이다. 즉, 위의 과정에서 `Survival 0`영역이 가득차 `GC`를 수행하고 살아남은 객체를
`Survival 1`영역으로 이동시킨다고 했는데, 이동을 하고 나면 `Survival 0`는 아무것도 없는 빈 상태가 된다. 추가로 `Eden`영역이 또 가득찬다면 `GC`를 수행하고
살아남은 객체는 비어있지 않은 `Survival 1`영역으로 이동하는 것이다. 이 과정이 반복해서 수행된다.(반대로 `Survival 1`이 가득차면 GC 수행하고 살아남은 객체 `Survival 0`로 이동)


이 과정이 반복되다가 객체의 Age 값이 특정 값 이상이되면 `Old Generation`영역으로 옮겨지는데 이 과정을 **Promotion**이라고 한다.
이 `Old Generation`영역이 가득차게되면 `GC`가 발생하는데 이는 `Major GC`라고 한다.

#### 이런 과정이 반복되면서 Garbage Collector가 메모리를 관리하는 것이다!!!

## Garbage Collector 종류

### 1. Serial GC
`GC`를 처리하는 스레드가 1개이다. CPU 코어가 적을 때 사용하는 방식이다.

`Mark-compact collection` 알고리즘을 사용한다.
![image](https://user-images.githubusercontent.com/60773356/126800307-7d1bfdab-80a0-4928-9010-c304f4c0c6ef.png)
* `Mark&Sweep`과정 이후에 `Compact`과정을 통해 `메모리 파편화`를 방지하는 알고리즘이다.

### 2. Parallel GC
`GC`를 처리하는 스레드가 여러개이다. 메모리가 충분하고 코어의 개수가 많을 때 사용하면 좋다.

아래 그림은 `Serial GC`와 `Parallel GC`의 스레드를 비교한 그림이다.
![image](https://user-images.githubusercontent.com/60773356/126800600-a3eeaffb-65c7-4ecf-970c-d413291338d5.png)


### 3. Concurrent Mark Sweep GC (CMS GC)
**Stop-The-World** : `GC`를 실행하기 위해 `JVM`이 애플리케이션 실행을 멈추는 것, `GC`를 실행하는 스레드를 제외한 스레드는 모두 작업을 멈추고 `GC` 완료 후에 다시 시작한다.

![image](https://user-images.githubusercontent.com/60773356/126801059-996e4bd0-6df8-46e9-9339-e4a248f8fe86.png)

쉽게 말해 `Stop-The-World`시간을 줄이는 GC이다. 애플리케이션의 응답 시간이 빨라야할 때 사용하면 좋다.

하지만 다른 `GC` 방식보다 메모리와 CPU를 더 많이 사용하고 `Compaction`단계가 제공되지 않는다는 단점이 존재한다.


### 4. G1 GC
![image](https://user-images.githubusercontent.com/60773356/126801483-edb2f6d2-e231-455e-bd29-3535b52f22d7.png)

각 영역을 `Region` 영역으로 나누고, `GC`가 일어날 때 전체 영역(Eden, Survival, Old generation)을 탐색하지 않는다.

`G1 GC`는 바둑판의 각 영역에 객체를 할당하고 `GC`를 실행한다. 해당 영역이 꽉 차면 다른 빈 영역에 객체를 할당하고 `GC`를 실행한다.

`Stop-The-World`이 짧고, `Compaction`을 사용한다는 장점이 존재한다.



#### 참고
* [네이버 D2](https://d2.naver.com/helloworld/1329)
