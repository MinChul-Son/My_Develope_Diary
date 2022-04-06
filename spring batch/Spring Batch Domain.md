# Spring Batch Domain 이해

## Job
- 배치 계층 구조에서 가장 상위의 개념, 하나의 배치 작업 자체를 의미
- `Job Configuration`을 통해 생성되는 객체 단위로 작업을 어떻게 구성하고 실행할 것인지 전체적으로 설정하고 명세한 객체(명세서, 설계도)
- `Batch Job`을 위한 최상위 인터페이스
- 여러 `Step`을 포함할 수 있는 컨테이너의 역할을 하며 최소 한개 이상의 `Step`을 가져야함

### 기본 구현체
- `SimpleJob`
	- 순차적으로 `Step`을 실행시키는 `Job`
	- 모든 `Job`에서 유용하게 사용할 수 있는 표준 기능을 갖고있음
- `FlowJob`
	- 특정한 조건과 흐름에 따라 `Step`을 구성하여 실행시키는 `Job`
	- `Flow`객체를 실행시켜 작업을 진행

![스크린샷 2022-04-04 오후 11 06 05](https://user-images.githubusercontent.com/60773356/161926592-9f417e7c-b07c-4e88-9c1c-df08155fd0f1.png)



![스크린샷 2022-04-04 오후 11 19 55](https://user-images.githubusercontent.com/60773356/161926611-287381c1-d9e8-47ab-8c1d-4996ce64c07c.png)

- `Job`을 실행하면 `SimpleJobBuilder`가 스프링 빈으로 생성된 `Step` 을 내부의 `steps`에 저장해둠

![스크린샷 2022-04-04 오후 11 23 13](https://user-images.githubusercontent.com/60773356/161926626-7fe11b1f-6a61-4b8b-863f-e8f794c96fdc.png)

- `build()`를 하게되면 `Job`을 생성하게 되는데 이 때 사용되는 구현체는 `SimpleJob`이다.
- `SimpleJob`의 `setSteps()`를 호출하며 `List<Step>`을 넘겨준다.


![스크린샷 2022-04-04 오후 11 24 20](https://user-images.githubusercontent.com/60773356/161926645-048f6ff6-400e-4293-914f-e49094301c71.png)

- `SimpleJob`은 마찬가지로 내부에 `step`를 저장하게 됨
- 최종적으로 `Job`이 `Step`을 담고있는 컨테이너의 역할


---

## JobInstance
- `Job`이 실행될 때 생성되는 `Job`의 논리적 실행 단위 객체로 고유하게 식별 가능한 작업 실행을 나타냄
- `Job`의 설정과 구성은 동일하지만 `Job`이 실행되는 시점에 처리하는 내용이 다르기 때문에 실행을 구분해야함
- 처음 시작하는 `Job + JobParameter`일 경우 새로운 `JobInstance`생성
- 이전과 동일한 `Job + JobParameter`으로 실행할 경우 이미 존재하는 `JobInstance` 반환
- `Job`과는 1:N 관계
- `JOB_NAME`과 `JOB_KEY`가 동일한 데이터는 중복 저장 불가

![스크린샷 2022-04-05 오후 2 38 26](https://user-images.githubusercontent.com/60773356/161926679-31c3501a-f3ac-41f8-8d69-6e731d10cc4b.png)

- `JobRepository`는 `Job Meta Data`를 데이터베이스에 저장하는 역할
	- 이 때 이미 존재하는 `JOB_NAME`과 `JOB_KEY`라면 이미 있는 `JobInstance` 반환
	- 그렇지 않으면 새로운 `JobInstance`생성


![스크린샷 2022-04-05 오후 7 54 24](https://user-images.githubusercontent.com/60773356/161926695-30d02fa3-291f-458c-b346-cb4bd6af452e.png)


![스크린샷 2022-04-05 오후 7 52 40](https://user-images.githubusercontent.com/60773356/161926704-bafd80cb-ad5d-4d88-89c3-fa2c341d9e37.png)

- `yml`에서 `spring.batch.job.enable=false`로 변경하여 스프링부트가 자동으로 `JobInstance`를 생성하는 것을 해제한 다음 `JobLauncher`를 통해 직접 `JobInstance`를 생성함
- 이 때 처음에는 `JobInstance`가 정상적으로 생성되지만 재실행하면 이미 동일한 `JobInstance`가 존재한다는 오류가 발생함


![스크린샷 2022-04-05 오후 7 55 15](https://user-images.githubusercontent.com/60773356/161926718-67b2971f-f1d0-4b70-8dc7-0ed41b173a9e.png)


다시 실행하려면 `JobParameter`를 이전과 다른 값으로 변경해야한다!
- `JOB_KEY`는 중복될 수 없다!


---

## JobParamter
- `Job`을 실행할 때 함께 포함되어 사용되는 파라미터를 가진 객체
- 하나의 `Job`에 존재할 수 있는 여러 `JobInstance`를 구분하기 위함
- `JobInstance`와 1:1 관계
- 사용 방법
	- 명령줄 인수: `java -jar LogBatch.jar requestDate=20220405`
	- 코드로 생성: `JobParameterBuilder`,  `DefaultJobParamterConverter` 사용
	- `SpEL`: `@Value("{jobParameter[requestDate]}")`
		- `@JobScope`, `@StepScope` 선언 필수
- `JOB_EXECUTION`과 1:N 관계

![스크린샷 2022-04-05 오후 8 32 00](https://user-images.githubusercontent.com/60773356/161926735-2f16b906-6074-4e68-ad54-86d1f9a0387b.png)


- `ParameterType`은 `Enum`으로 제공됨


### Job에서 JobParameter를 참조하는 방법
![스크린샷 2022-04-05 오후 9 18 55](https://user-images.githubusercontent.com/60773356/161926752-0f826902-e882-4508-a1dd-c36045aec29a.png)

- `Job`내부의 `Step`에서 `tasklet` 내부의 람다를 살펴보면 첫번째 인수는 `StepContribution`이다.
- 이 `StepContribution` 내부에 `StepExecution`이 존재하고 그 내부에는 `JobExecution`이 존재한다.

![스크린샷 2022-04-05 오후 9 20 52](https://user-images.githubusercontent.com/60773356/161926764-6e31d5c4-5cfb-4227-8ade-f65e04d9e976.png)

- `JobExecution` 내부에는 해당 `Job`과 관련된 값들이 존재하는데 여기에 `JobParameters`가 있다.


![스크린샷 2022-04-05 오후 9 23 38](https://user-images.githubusercontent.com/60773356/161926777-d37b552a-420b-489e-a82f-7e56ecd29712.png)

람다의 두번쨰 인자인 `ChunkContext` 내부에도 `StepContext`가 있고 그 내부에 `StepExecution`이 존재하기 때문에 동일하게 참조를 할 수 있다.


![스크린샷 2022-04-05 오후 9 27 00](https://user-images.githubusercontent.com/60773356/161926789-4051a419-b5fa-4f92-bada-5253fa24361d.png)

- 참조를 통해 `JobParameters`를 가져온 뒤, 모든 `JobParameter`가 담겨있는 `Map`을 꺼내어 사용할 수도 있고 해당하는 타입으로 `key`를 통해 하나만 조회할 수도 있음


![스크린샷 2022-04-05 오후 9 30 06](https://user-images.githubusercontent.com/60773356/161926804-faa4f3ee-299f-4bee-89ec-86ff092356c1.png)

- `ChunkContext`를 사용해 `JobParamter`를 꺼내올 수 있음
- `StepContribution`과 같은 참조를 통해 꺼내올 수도 있고 `StepContext`에서 바로 `Map`을 꺼내올 수도 있음
