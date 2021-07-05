# 테코톡 스터디 : Spring IoC/DI
* 10분 테코톡을 시청하고 작성하였습니다.
* [영상링크](https://www.youtube.com/watch?v=_OI9mKuFb7c&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)
* 참고용 : 이전에 영한님 강의듣고 정리해놓은 [IoC/DI](https://github.com/MinChul-Son/My_Develope_Diary/blob/main/Spring/IoC%26DI.md)

## 스프링의 대 삼각형
* 스프링의 가장 밑바탕이 되는 3가지
* ![image](https://user-images.githubusercontent.com/60773356/124468158-bafe5480-ddd3-11eb-9036-388c2a232090.png)
  - IoC/DI ==> **가장 기본**
  - AOP
  - PSA

## IoC / DI
* Spring Framework의 근간
* Object의 생명주기와 의존관계에 대한 프로그래밍 모델
* 유연하고 확장성이 뛰어난 코드를 만들 수 있게 해주는 프로그래밍 모델
> > "토비의 스프링"에서 발췌

### 관심사의 분리
* **유연하고 확장성이 뛰어나다.**
  - (= 변경이 있을 때 수정이 쉽다.)
  - (= 수정할 부분만 수정하면 된다.)
  - (= 관심사의 분리가 잘 이루어졌다.)
---------------------------------------

### 전략 패턴(Strategy Pattern)
* 관심사의 분리를 잘 설명할 수 있는 패턴
* 객체들이 할 수 있는 행위 각각에 대해 전략 클래스를 생성하고, 유사한 행위들을 캡슐화 하는 인터페이스를 정의하여, **객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장**하는 방법
* 객체가 할 수 있는 행위들 각각을 전략으로 만들어 놓고, 동적으로 행위의 수정이 필요한 경우 전략을 바꾸는 것만으로 행위의 수정이 가능하도록 만든 패턴
* [전략 패턴이 무엇인가요?](https://github.com/MinChul-Son/My_Develope_Diary/blob/main/Strategy%20Pattern.md)
```java
public class Car {
  private MovingStrategy movingStrategy;
  private int distance;
  
  public Car(MovingStrategy movingStrategy) {
    this.movingStrategy = movingStrategy;
    this.distance = 0;
  }
  void move() {
    if (movingStrategy.isMovable()) {
      distance++;
    }
  }
}
```
* 전략 패턴 코드 예시를 가지고 생각해보자.
* Car 객체의 입장에서는 직접 전략을 선택하는 것이 아니기 때문에 자신이 어떤 전략을 사용할지도 알지 못함.
* 즉, **제어에 대한 주도권이 Car객체 스스로에게 없음**
* 제어권은 Car객체를 사용하는 클라이언트에게 있다.

#### 클라이언트
```java
public class Racing {
  private static final int NUM_OF_CARS = 10;
  private List<Car> cars = new ArrayList<>();
  
  public Racing() {
    MovingStrategy strategy = new RandomMovingStrategy();
    
    for (int i = 0; i < NUM_OF_CARS; i++) {
      cars.add(new Car(strategy));
    }
  }
  public void move() {
    cars.forEach(Car::move);
  }
}
```
* Car 객체에 대한 모든 제어를 클라이언트가 하고있음.
* 하지만 이 또한 관심사의 분리가 완벽하게 이루어지지 않음.
* Racing은 Car가 어떻게 움직일 것인가에만 관심을 두어야지 Car의 생성에 대해서는 관심이 없다. 따로 분리하는게 맞음.
* Racing은 Car의 움직임에 집중하고, Car의 생성은 CarFactory를 만들어 관심사를 분리하면 더 좋은 코드가 될 것이다.

### 제어의 두 가지 관점!
1. 어떤 연관관계를 맺으며 생성될 것인가?
2. 어떻게 사용될 것인가?

##### Car의 MovingStrategy를 변경하고 싶다?
* **MovingStrategy의 새로운 구현체를 만들어 CarFactory에게 넘겨주면 된다.**

##### Racing의 방식을 변경하고 싶다?
* **Racing 내부의 코드만 변경하면 된다.**

------------------------------------------------------
## Spring IoC/DI
### 제어의 두 가지 관점
* 어떤 연관관계를 맺을 것인가?
  - 팩토리 -> 빈 컨테이터  
* 어떻게 사용될 것인가?
  - 클라이언트 -> 스프링 내부 코드

## 결론
* IoC/DI는 사실 자연스럽게 사용하고 있던 것
  - 우리는 컨트롤러, 서비스, 스프링 빈에 대한 구현을 하고 있고 제어의 두 가지 관점에 대한 코드들은 모두 스프링이 내부적으로 처리해줌
* 관심사의 분리를 통해 유연하고 확장성이 쉬운 코드를 만들다보니 자연스럽게 만들어진 프로그래밍 모델.

