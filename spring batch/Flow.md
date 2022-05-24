# Flow

## FlowJob
- `Step`을 순차적으로만 구성하는 것이 아니라 특정한 상태에 따라 흐름을 전환하도록 구성할 수 있음
	- `Step`이 실패해도 `Job`은 성공하도록 해야하는 경우
	- 특정 `Step`이 성공했을 때 다음 `Step`을 구분해서 실행해야하는 경우
- `FlowJobBuilder`로부터 생성
- 내부적으로 `SimpleFlow`객체를 포함하고 있어 `Job` 실행시 호출함
- 조건적 흐름
- `JobBuilderFactory` -> `JobBuilder` -> `JobFlowBuilder` -> `FlowBuilder` -> `FlowJob`
- `FlowBuilder`의 `on()` 메서드를 호출하면 `TransitionBuilder`가 작동해 `Step`간의 조건부 전환을 구성할 수 있음
	- `to()` : 특정 상태일 때 특정 `Step`으로 이동
	- `stop()`, `fail()`, `end()`, `stopAndRestart()`: 특정 상태일 때 `Flow`종료
	- 반환값이 `FlowBuilder`이기 때문에 다시 다른 `Step`이나 `Flow`를 실행 시킬 수 있음

![스크린샷 2022-05-11 오후 4 38 34](https://user-images.githubusercontent.com/60773356/168428674-f3a218f9-9adf-47e7-8f27-92d454e1bb31.png)

<br>
<br>


![스크린샷 2022-05-11 오후 5 10 33](https://user-images.githubusercontent.com/60773356/168428678-49fa3d63-a10e-4f56-a7fa-67613c3e4e5c.png)
- `step1`이 성공했을 시 `step2` 실행
- `step1`이 실패했을 시 `step3` 실행
	- `step1`이 실패하더라도 전체 `Job`은 `COMPLETED`
- 조건에 따라 분기되기 때문에(실패에 대한 대응이 되어있기 때문에) `Job`이 성공으로 처리
	- `try-catch`와 비슷한 맥락으로 생각하자

<br>

---

## API

### start(Flow) 
- 처음으로 실행될 `Flow`설정
- `JobFlowBuilder` 반환
	- 파라미터로 `Step` 객체가 들어오면 `SimpleJobBuilder`가 반환

<br>

### next(Step or Flow or JobExecutionDecider)
- 다음에 실행될 `Step`, `Flow`, `JobExecutionDecider`를 지정하는 설정


`on()`을 사용하지 않고 `start`, `next`를 사용해서 `Job`을 구성하면 내부 `Step`이 하나라도 실패하면 전체 `Job`이 실패한다.

<br>

---

## SimpleFlow
- `Spring Batch`가 제공하는 `Flow` 구현체
- `Step`, `Flow`, `JobExecutionDecider`를 담고 있는 `State`를 실행시킴
	- `State`는 `SimpleFlow`가 가지고 있는 속성
- `FlowBuilder`를 통해 생성, `Transition`과 조합해 여러 `Flow` 또는 중첩 `Flow`로 `Job`을 구성할 수 있음

<br>


### State
-  `Step`, `Flow`, `JobExecutionDecider`를 저장하고 있는 객체
- `Flow`를 구성하면 내부적으로 생성되고 `Transition`과 연동
- 내부 `handle()` 메서드를 실행하고 `FlowExecutionStatus`를 반환
	- 마지막 실행상태가 `FlowJob`의 최종 상태
	- `on()` 메서드로 넘어온 패턴과 매칭여부를 판단함
	- `SimpleFlow`가 `State`를 실행시켜 `Step`이나 `Flow`를 실행시키는 것
- `SimpleFlow`는 `StateMap`에 저장된 모든 `State`의 `handle()`메서드를 통해 모든 `Step`이 실행되도록 함
	- 이 때 호출되는 `State`가 어떤 타입인지 알지 못함 -> 상태 패턴
	- 상태값만 얻어옴

<br>

## FlowStep
- `Step`내에 `Flow`를 할당해 실행시키는 도메인 객체
- `Flow`의 최종 상태값에 따라 `FlowStep`의 `BatchStatus`와 `ExitStatus`가 결정
- `StepBuilder`하위의 `FlowStepBuilder`로부터 생성
