# Job

## 배치 초기화 설정

### 1. JobLauncherApplicationRunner
- `Spring Batch` 작업을 시작하는 `ApplicationRunner`로 `BatchAutoConfiguration`에서 생성됨
- 빈으로 등록된 모든 `Job`을 실행시킴

### 2. BatchProperties
- `Spring Batch`의 환경 설정 클래스
- `Job` 이름, 스키마 초기화 설정, 테이블 `prefix`등의 값 설정
- `yml`에서 설정함

### 3. Job 실행 옵션
- 지정한 `Job`만 실행하도록 할 수 있음
- `program arguments`로 `job`이름 지정
	- `--job.name=hello`
	- 여러 `job`을 실행하려면 쉼표로 구분



## JobBuilderFactory / JobBuilder
스프링 배치는 `Job`과 `Step`을 쉽게 생성, 설정할 수 있게 빌더 클래스를 제공

- `JobBuilderFactory`
	- `JobBuilder`를 생성하는 팩토리 클래스
	

- `JobBuilder`
	- `Job`을 구성하는 설정 조건에 따라 두 개의 하위 빌더 클래스를 생성하고 `Job` 생성 역할을 위임
	- `SimpleJobBuilder`
		- `SimpleJob`을 생성하는 클래스
		- `Job` 실행과 관련된 `API` 제공
	- `FlowJobBuilder`
		- `FlowJob`을 생성하는 클래스
		- 내부적으로 `FlowBuilder`를 반환해 `Flow`실행과 관련된 `API` 제공

![스크린샷 2022-04-29 오후 3 55 09](https://user-images.githubusercontent.com/60773356/167344483-0e3f5286-7926-41f1-ae12-70e139602aea.png)





## SimpleJob
- `SimpleJob`은 `Step`을 실행시키는 `Job` 구현체로서 `SimpleJobBuilder`에 의해 생성됨
- 여러 단계 `Step`으로 구성 가능하고, 순차적으로 실행
- 모든 `Step`이 성공해야  `Job`이 성공

> `JobBuilderFactory` -> `JobBuilder` -> `SimpleJobBuilder` -> `SimpleJob`


### API
- `get`
- `start(Step)`
- `next(Step)`
- `incrementer(JobParamtersIncrementer`
	- `JobParameter`의 값을 자동 증가시켜 다음에 사용될 `JobParameters `를 리턴
	- 기존의 `JobParameter`  변경없이 `Job`을 여러번 시작하고자 할 때
	- `RunIdIncrementer` 구현체 제공, 필요시 인터페이스 직접 구현 가능
- `preventRestart(boolean)`
	- `Job`의 재시작 가능 여부(default = true)
	-  실패하더라도 재시작이 안되며 재시작을 하려고하면 `JobRestartException` 발생
- `validator(JobParameterValidator`
	- `Job` 실행에 필요한 파라미터를 검증하는 역할
	- `DefaultJobParametersValidator` 구현체를 제공, 복잡한 조건이 있을 시 인터페이스 직접 구현 가능
	- `requiredKeys`에 필수 키값, `optionalKeys`에 필수가 아닌 키값
	- 필수 키가 없으면 `JobParametersInvalidException` 발생
	- `JobRepository`의 기능이 수행되기 이전에 한번 검증, `Job`이 실행되기 이전에 한번 검증 -> 총 2번의 검증을 거침
- `listener(JobExecutionListener)`
	- 특정 시점에 콜백을 제공받도록하는 API
- `build`
























