# 🌱 Step

## StepBuilderFactory
- `StepBuilder`를 생성하는 팩토리 클래스

<br>

## StepBuilder
- `Step`을 구성하는 설정 조건에 따라 하위 빌더 클래스를 생성하고 `Step`생성을 위임
- `TaskletStepBuilder`
- `SimpleStepBuilder`
- `PartitionStepBuilder`
- `JobStepBuilder`
- `FlowStepBuilder`

<br>

### StepBuilder API
- `tasklet(tasklet())` -> `TaskletStepBuilder`
- `chunk(chunkSize)` -> `SimpleStepBuilder`
- `chunk(completionPolicy` -> `SImpleStepBuilder`
- `partitioner(stepName, partitioner)` -> `PartitionStepBuilder`
- `partitioner(step` -> `PartitionStepBuilder`
- `job(job)` -> `JobStepBuilder`
- `flow(flow)` -> `FlowStepBuilder`

<br>
<br>

---

## TaskletStep
- `Step`의 구현체, `Tasklet`을 실행시키는 도메인 객체
- `RepeatTemplate`를 사용해 `Tasklet`의 로직을 트랜잭션 경계 내에서 반복 실행
	- 트랜잭션 처리가 별도로 필요없음, 스프링 배치 내부적으로 처리되어있음
- `Task`기반과 `Chunk` 기반으로 나누어 실행

<br>

### Task 기반
- 단일 작업 기반으로 처리되는 것이 더 효율적인 경우
- `Tasklet` 구현체를 만들어 사용
- `Chunk` 기반에 비해 더 복잡한 구현 필요

<br>

### Chunk 기반
- 대량 처리를 하는 경우 효과적
- `ItemReader`, `ItemProcessor`, `ItemWriter`를 사용
-  `ChunkOrientedTasklet` 구현체가 제공됨

<br>

### API
- `tasklet()`: `Tasklet` 클래스 설정
	- `Chunk` or `Task`
- `startLimit()`: `Step`의 실행 횟수 설정, 초과시 예외 발생
- `allowStartIfComplete()`: `Step`의 성공, 실패 여부에 상관없이 항상 `Step`을 실행하기 위한 설정
- `listener()`: 특정 시점에 콜백을 위한 리스너 설정
- `build()`: `TaskletStep`을 생성

<br>

### tasklet()
- `Tasklet` 타입의 클래스를 설정
- 익명 클래스 or 구현체를 만들어 사용
- `TaskletStepBuilder`가 반환
- `Step`에 하나의 `Tasklet` 설정 가능, 두개 이상을 설정하면 마지막 설정 객체만 실행됨

<br>

### startLimit()
- `Step`의 실행 횟수를 조정
- `Step`마다 설정 가능
- 초과해서 실행하면 `StartLimitExceededException` 발생
- 실행하면  `JobExecution`은 계속 생성되지만 `StepExecution`은 설정된 값 이상으로 생성되지 않음

<br>

### allowStartIfComplete()
- 재시작 가능한 `Job`에서 `Step`의 성공 여부와 관계없이 항상 실행하기 위한 설정
- 실행마다 유효성을 검증하는 `Step`, 사전 작업 `Step`에 사용하면 유용한 옵션
- `true`로 설정되면 `COMPLETED` 상태가 되더라도 항상 실행

<br>
<br>

---

## JobStep
- `Job`에 속하는 `Step` 중 외부의 `Job`을 포함하고 있는 `Step`
	- 즉, `Step`안에 또다른 `Job`이 존재
- 모든 메타데이터는 기본 `Job`과 외부 `Job`별로 각각 저장
- 내부의 `Job`이 실패하면 외부의 `Job`도 실패
- `JobParamtersExtractor`는 `ExecutionContext`내의 `key`를 찾아오는 역할
	- `Spring Bean`으로 등록하지 않아도됨


