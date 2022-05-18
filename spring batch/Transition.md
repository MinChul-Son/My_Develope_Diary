# Transition
- 조건에 따라 흐름을 전환, 전이하는 개념을 의미
- `Flow`내 `Step`의 조건부 전환
- `on()` 메서드를 호출하면 `TransitionBuilder`가 반환되어 `Transition Flow`를 구성 가능
- 구체적인 것이 덜 구체적인 것보다 우선
	- 특정 `Step`에 대해 `on("*")`과 `on("FAILED")` 2개가 정의되어 있다면 `on("FAILED")`가 우선

<br>

## 배치 상태 유형 

<br>

### BatchStatus
- `JobExecution`과 `StepExecution`의 속성으로 `Job`, `Step` 종료 후 최종 결과 상태
- `SimpleJob`은 마지막 `Step`의 `BatchStatus`값을 반영
- `FlowJob`은 마지막 `Flow`의 `FlowExecutionStatus`의 값을 반영
- `COMPLETED`, `STARTING`, `STARTED`, `STOPPING`, `STOPPED`, `FAILED`, `ABANDONED`, `UNKNOWN`
	- `ABANDONED` -> 실패했지만 재시작시에 건너뜀

<br>

### ExitStatus
- `JobExecution`과 `StepExecution`의 속성으로 `Job`, `Step`의 실행 후 어떤 상태로 종료되었는지에 대한 상태
- 기본적으로 `BatchStatus`와 동일한 값으로 설정
-  `SimpleJob`은 마지막 `Step`의 `ExitStatus`값을 반영
- `FlowJob`은 마지막 `Flow`의 `FlowExitStatus `의 값을 반영
- `UNKNOWN`, `EXECUTING`, `COMPLETED`, `NOOP`, `FAILED`, `STOPPED`

<br>

### FlowExecutionStatus
- `FlowExecution`의 속성으로 `Flow` 실행 후 최종 결과 성태
- `Flow`내부의 `Step`의 `ExitStatus`값을 `FlowExecutionStatus`로 저장
- `FlowJob`의 배치 결과 상태에 관여
- `COMPLETED`, `STOPPED`, `FAILED`, `UNKNOWN`

<br>

---

<br>

## API

<br>

### on(String pattern)
- 어떤 상태로 종료되었을 때 실행될지를 설정(`ExitStatus`와 매칭
- on(“FAILED”) 
	- `FAILED`라면 지정된 다음 `Step`을 실행(`to()`에서 정의)
- 매칭되지 않으면 예외가 발생하여 `Job`이 실패함
- 2가지 특수문자 허용
	- `*`: 0개 이상의 문자와 매칭, 모든 `ExitStatus`와 매칭
	- `?`: 정확히 1개 문자와 매칭
		- `C?t`는 `cat`과 매칭 O, `Caat`과는 매칭 X

<br>

### to(Step or Flow or JobExecutionDecider)
- 다음으로 실행할 단계를 지정

<br>

### from()
- `Transition`을 추가 정의함 

<br>

### stop()
- `FlowExecutionStatus`가 `STOPPED`상태로 종료되는 `transition`

<br>

### fail()
- `FlowExecutionStatus`가 `FAILED`상태로 종료되는 `transition` 

<br>

### end()
- `FlowExecutionStatus`가 `COMPLETED`상태로 종료되는 `transition`

<br>

### stopAndRestart(Step or Flow or JobExecutionDecider)
- `stop()`과 기본 흐름 동일
- `step`을 중단, 중단 이전의 `Step`만 `COMPLETED`로 저장되고 실행되지 않은 이후의 `step`은 `STOPPED` 상태로 `Job`종료
- 이후에 `Job`이 재시작되었을 때 `STOPPED`된 `step`부터 시작
