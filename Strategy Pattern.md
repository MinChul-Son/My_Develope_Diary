# 전략 패턴(Strategy Pattern)
* 관심사의 분리를 잘 설명할 수 있는 패턴
* 객체들이 할 수 있는 행위 각각에 대해 전략 클래스를 생성하고, 유사한 행위들을 캡슐화 하는 인터페이스를 정의하여, **객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장**하는 방법
* 객체가 할 수 있는 행위들 각각을 전략으로 만들어 놓고, 동적으로 행위의 수정이 필요한 경우 전략을 바꾸는 것만으로 행위의 수정이 가능하도록 만든 패턴

## 전략 패턴 예시 코드
* 내부에 move메서드가 있는 Movable이라는 인터페이스가 있다고 가정해보자.
* ![image](https://user-images.githubusercontent.com/60773356/124469845-d702f580-ddd5-11eb-9b82-9dd53f0409a9.png)
* 이 인터페이스를 구현한 Car와 Airplane 클래스를 구현하였다.
```java
//자동차
public class Car implements Movable{
  public void move(){
    System.out.println("차는 도로에서 움직여요!");
  }
}

//비행기
public class Airplane implements Movable{
  public void move(){
    System.out.println("비행기는 하늘에서 움직여요!");
  }
}
```
* 현재까지는 아무 문제가 없다. 하지만 만약 하늘을 나는 자동차가 새로 만들어졌다면?!?!
```java
public void move(){
    System.out.println("차는 도로에서 움직여요!");
  }
```
* Car의 move 메서드를 위와 같이 수정해주어야한다. 하지만 이는 **OCP위반!!**
* 또한 여러 Movable 구현체가 생겨났을때 동일한 move메서드를 다른 클래스에서 작성하여 메서드의 중복이 발생한다.

------------------------
### 여기에 전략 패턴을 적용해보자
* MovableStrategy라는 인터페이스를 생성한다.
* ![image](https://user-images.githubusercontent.com/60773356/124470839-18e06b80-ddd7-11eb-8829-2f552fe99435.png)
* 이를 구현한 구현체인 RoadStrategy(도로로 이동), SkyStrategy(하늘로 이동)를 구현한다.
```java
//도로
public class RoadStrategy implements MovableStrategy{
    public void move(){
        System.out.println("도로에서 움직여요!");
    }
}
//하늘
public class SkyStrategy implements MovableStrategy{
    public void move(){
        System.out.println("하늘에서 움직여요!");
    }
}
```
* 이제 이렇게 구현된 전략을 각 운송 수단 객체의 이동 전략으로 선택하기만 하면 되는 것이다!
```java
public class Car {
  private MovableStrategy movableStrategy;
  public Car(MovableStrategy movableStrategy) {
    this.movableStrategy = movableStrategy;
  }
}
// 메인 클래스
Car car = new Car(new RoadStrategy);
```
* 만약 하늘을 나는 자동차가 개발되더라도 전략을 SkyStrategy로 변경만해주면 간단하게 해결된다.
