# CountDownLatch
[공식문서](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CountDownLatch.html)

공식 문서에서는 `CountDownLatch`를 이렇게 설명하고 있다.
> A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.

쉽게 설명하자면, **어떤 쓰레드가 다른 쓰레드들의 작업이 완료될 때까지 대기하도록 동기화** 시켜주는 것이다.

이렇게 이야기하면 언제 필요한지 확 와닿지 않을 수 있다.

아래 그림을 살펴보자.

![image](https://user-images.githubusercontent.com/60773356/139405505-76e0f4e5-7aab-4d92-a180-1b7762238fff.png)
- `Main Thread` 안에 두개의 쓰레드가 존재
- 쉽게 생각해 `Main Thread`는 `Java`의 `main`함수라고 생각하면 된다.

여기서 만약 `Main Thread`가 종료된다면 `Thread 1`, `Thread 2`가 실행 상태에 상관없이 모두 종료된다.

`Thread 1`이 어떤 로직을 실행하고 있더라도 말이다.

![image](https://user-images.githubusercontent.com/60773356/139407052-aa357e29-eba2-4235-9bc6-5933af208cb7.png)
- 각 쓰레드에서 임의로 어떤 로직을 수행하고있다는 가정을 위해 `Thread.sleep`을 사용하여 1초 동안 지연시켰다.

여기서 `main Thread`는 `thread1`과 `thread2`를 `start()`한 후 자신의 작업이 완료되었기에 종료된다.

![image](https://user-images.githubusercontent.com/60773356/139407100-4d078d18-8d96-4e4e-b301-d26247984679.png)

테스트 결과를 살펴보면 `Thread1`과 `Thread2`의 작업이 완료되기 전에 `Main Thread`가 종료된 것을 확인할 수 있다.

이럴 때, 유용하게 사용할 수 있는 것이 바로 `CountDownLatch`이다.

OS에서 `block` 상태로 만들고 다시 깨우는 것과 비슷한 형식으로 동작한다.

사용법은 간단하다.

```java
CountDownLatch countDownLatch = new CountDownLatch(5);
```
를 통해 `CountDownLatch` 인스턴스를 생성하는데 이 때 전달되는 값은 작업의 수라고 생각하면된다.

이 때 각 작업마다 `countDownLatch.countDown()`를 호출한다.

호출될 때마다 생성할 때 입력된 Latch(여기서는 5)의 값이 1씩 감소하고 최종적으로 0이 되면 다음 코드를 실행한다.

상위 쓰레드에서 `countDownLatch.await()`를 걸어 대기하도록 만들고 하위 쓰레드에서 `countDown()`을 통해 `Latch`를 줄여 0이되면 다음 코드로 넘어가는 것이다.

![image](https://user-images.githubusercontent.com/60773356/139408420-60118a46-4c76-4c74-a8a6-35df83eceb39.png)
>예시 코드를 위해 위와 같이 쓰레드를 마구잡이로 하드 코딩하였다. 실제 사용에서는 별도의 클래스를 만들고 분리하여 사용하자!

![image](https://user-images.githubusercontent.com/60773356/139408484-22848819-eb6b-4a39-8637-f81d73bddef0.png)

테스트 결과를 보면 알 수 있듯이 모든 쓰레드의 작업이 종료된 후에 `Main Thread`가 종료되었음을 확인할 수 있다.

하지만 `countDown`이 호출되지 않는다면 **상위 쓰레드는 무한히 기다릴 수 있다는 점**을 주의하자!!

이를 위해 `timeout` 시간을 지정하는 `await` 메소드도 별도로 존재한다.

![image](https://user-images.githubusercontent.com/60773356/139409084-cf1f260e-2a42-430d-a81e-8f5ca36f5b8d.png)

병렬 쓰레드, 멀티 쓰레드 시에 유용하게 사용될 수 있을 듯하다!




