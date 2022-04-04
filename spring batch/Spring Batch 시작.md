# Spring Batch 시작하기
## Spring Batch 활성화
- `@EnableBatchProcessing`: 스프링 배치가 작동하기 위해 선언해야 하는 애노테이션
	- 총 4개의 설정 클래스를 실행시켜 `Spring Batch`의 모든 초기화 및 실행 구성이 이뤄짐
	- 이 애노테이션을 추가하면 설정 클래스들이 실행되어 스프링 빈으로 등록된 모든 `Job`을 검색해 초기화와 동시에 수행하도록 구성함


### Spring Batch Initialization Class
- `BatchAutoConfiguration`
	- 스프링 배치가 초기화될 때 자동으로 실행되는 설정 클래스
	- `Job`을 수행하는 `JobLauncherApplicationRunner`를 생성
- `SimpleBatchConfiguration`
	- `JobBuilderFactory`와 `StepBuilderFactory` 생성
	- `Spring Batch`의 주요 구성 요소 생성 -> 프록시 객체
- `BatchConfigurerConfiguration`
	- `BasicBatchConfigurer`
		- `SimpleBatchConfiguration`에서 생성한 프록시 객체의 실제 `target` 객체를 생성하는 설정 클래스
		- 스프링 빈으로 `DI`받아 주요 객체를 참조할 수 있음
	- `JpaBatchConfigurer`
		- `JPA` 관련 객체를 생성하는 설정 클래스
	- 추가로  `BatchConfigurer` 인터페이스를 구현해 커스텀하게 사용할 수도 있음


### 동작 순서
1. `@EnableBatchProcessing`
2. `SimpleBatchConfiguration`
3. `BatchConfigurerConfiguration`
4. `BatchAutoConfiguration`


## Job, Step의 간략한 흐름
![스크린샷 2022-03-25 오후 6 25 29](https://user-images.githubusercontent.com/60773356/161501239-f0191c80-20df-43b7-bb0f-794579e65a67.png)

- `Job`: 일
- `Step`: 일의 단계
- `Tasklet`: 작업 내용


## Spring Batch Meta Data
- 스프링 배치의 실행 및 관리를 위한 목적으로 여러 도메인들의 정보를 저장, 업데이터, 조회할 수 있는 스키마 제공
- 과거와 현재의 실행에 대한 세세한 정보, 실행에 대한 성공/실패 여부 등을 관리하여 리스크 발생시 빠른 대체 가능
- DB와 연동할 경우 필수적으로 메타 테이블이 생성되어야함

### 스키마 생성 설정
- 수동 생성 - 쿼리 복사 후 직접 실행
- 자동 생성 - `spring.batch.jdbc.initialize-schema`
	- `ALWAYS`
		- 스크립트 항상 실행
		- `RDBMS` 설정이 되어 있을 경우 내장 `DB`보다 우선 실행
	- `EMBEDDED`
		- 내장 `DB`일 때만 실행되며 스키마 자동생성, `default` 설정
	- `NEVER`
		- 스크립트 실행 안함
		- 내장 `DB`일 경우 오류 발생


<img width="1531" alt="스크린샷 2022-03-31 오후 10 59 33" src="https://user-images.githubusercontent.com/60773356/161501257-f922a824-b06e-4514-ae87-e7d4a0b6cd25.png">



### Job 관련 테이블
- `BATCH_JOB_INSTANCE`
	- `Job`이 실행될 때 `JobInstance` 정보가 저장되며 `job_name`과 `job_key`를 키로 하여 하나의 데이터가 저장
	- 칼럼
		- `JOB_INSTANCE_ID`: 기본키
		- `VERSION`: 업데이트될 때마다 1씩 증가
		- `JOB_NAME`: `Job`의 이름
		- `JOB_KEY`: `job_name`과 `jobParameter`를 합쳐 해싱한 값
	- 동일한 `job_name`과 `job_key`로 중복 저장될 수 없음
- `BATCH_JOB_EXECUTION`
	- `Job`의 실행정보 저장되며 생성, 시작, 종료 시간, 실행 상태, 메시지 등을 관리
	- 칼럼
		- `Job_EXECUTION_ID`: 기본키, `JOB_INSTANCE`와 일대다
		- `VERSION`: 업데이트될 때마다 1씩 증가
		- `JOB_INSTANCE_ID`: `JOB_INSTANCE `의 키
		- `CREATE_TIME`: 실행이 생성된 시점을 `TimeStamp` 형식으로 기록
		- `START_TIME`: 실행이 시작된 시점을 `TimeStamp` 형식으로 기록
		- `END_TIME`: 실행이 종료된 시점을 `TimeStamp`으로 기록, `Job` 실행 도중 오류가 발생해서 중단된 경우 값이 저장되지 않을 수 있음
		- `STATUS`: 실행 상태를 저장 (COMPLETED, FAILED, STOPPED…)
		- `EXIT_CODE`: 실행 종료코드를 저장 (COMPLETED, FAILED…)
		- `EXIT_MESSAGE`: `Status`가 실패일 경우 실패 원인 등의 내용을 저장
		- `LAST_UPDATED`: 마지막 실행 시점을 `TimeStamp` 형식으로 기록

- `BATCH_JOB_EXECUTION_PARAMS`
	- `Job`과 함께 실행되는 `JobParameter`정보를 저장
	- 칼럼
		- `JOB_EXECUTION_ID`: 기본키, `JOB_EXECUTION` 과는 일대다 관계
		- `TYPE_CD`: `STRING`, `LONG`, `DATE`, `DUBLE` 타입정보
		- `KEY_NAME`: 파라미터 키 값
		- `STRING_VAL`: 파라미터 문자 값
		- `DATE_VAL`: 파라미터 날짜 값
		- `LONG_VAL`: 파라미터 `LONG` 값
		- `DOUBLE_VAL`: 파라미터 `DOUBLE` 값
		- `IDENTIFYING`: 식별여부 (TRUE, FALSE)

- `BATCH_JOB_EXECUTION_CONTEXT`
	- `Job`의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화(JSON)해여 저장
	- `Step`간의 공유 가능
	- 칼럼
		- `JOB_EXECUTION_ID`: 기본키, `JOB_EXECUTION` 마다 각각 생성
		- `SHORT_CONTEXT`: `JOB` 의 실행 상태정보, 공유데이터 등의 정보를 문자열로 저장
		- `SERIALIZED_CONTEXT`: 직렬화된 전체 컨텍스트



### Step 관련 테이블
- `BATCH_STEP_EXECUTION`
	- `Step`의 실행정보가 저장되며 생성, 시작, 종료시간, 실행상태, 메시지 등을 관리
	- 칼럼
		- `STEP_EXECUTION_ID`: 기본 키
		- `VERSION`: 업데이트 될 때마다 1씩 증가
		- `STEP_NAME`: `Step` 을 구성할 때 부여하는 이름
		- `JOB_EXECUTION_ID`: `JobExecution` 과는 일대다 관계
		- `START_TIME`: 실행이 시작된 시점을 `TimeStamp` 형식으로 기록
		- `END_TIME`: 실행이 종료된 시점을 `TimeStamp`으로 기록하며 `Job` 실행 도중 오류가 발생해서 중단된 경우 값이 저장되지 않을 수 있음
		- `STATUS`: 실행 상태를 저장 (COMPLETED, FAILED, STOPPED…)
		- `COMMIT_COUNT`: 트랜잭션 당 커밋되는 수를 기록
		- `READ_COUNT`: 실행시점에 `Read`한 `Item` 수를 기록
		- `FILTER_COUNT`: 실행도중 필터링된 `Item`수를 기록
		- `WRITE_COUNT`: 실행도중 저장되고 커밋된 `Item`수를 기록
		- `READ_SKIP_COUNT`: 실행도중 `Read`가 `Skip`된 `Item`수를 기록
		- `WRITE_SKIP_COUNT`: 실행도중 `write`가 `Skip`된 `Item`수를 기록
		- `PROCESS_SKIP_COUNT`: 실행도중 `Process`가 `Skip`된 `Item` 수를 기록
		- `ROLLBACK_COUNT`: 실행도중 `rollback`이 일어난 수를 기록
		- `EXIT_CODE`: 실행 종료코드를 저장 (COMPLETED, FAILED…)
		- `EXIT_MESSAGE`: `Status`가 실패일 경우 실패 원인 등의 내용을 저장
		- `LAST_UPDATED`: 마지막 실행 시점을 `TimeStamp` 형식으로 기록

- `BATCH_STEP_EXECUTION_CONTEXT`
	- `Step`의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화(JSON)해서 저장
	- `Step`별로 저장되며 `Step`간 서로 공유할 수 없음
	- 컬럼
		- `STEP_EXECUTION_ID`: 기본키, `STEP_EXECUTION` 마다 각 생성
		- `SHORT_CONTEXT`: `STEP` 의 실행 상태정보, 공유데이터 등의 정보를 문자열로 저장
		- `SERIALIZED_CONTEXT`: 직렬화된 전체 컨텍스트







