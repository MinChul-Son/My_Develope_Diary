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


---

## JobExecution
- `JobInstance`에 대한 한번의 시도를 의미하는 객체로 `Job` 실행 중에 발생한 정보를 저장하는 객체
- `FAILED`나 `COMPLETED`등의 실행 결과 상태를 가지고 있음
	- `COMPLETED`라면 실행 완료로 판단해 재실행 풀가
	- `FAILED`라면 실행이 완료되지 않은 것으로 판단해 재실행 가능
		- 이 때는 `JobParameter`가 동일하더라도 계속 실행 가능
		- 즉, `COMPLETED`가 될 때까지 하나의 `JobInstance`내에서 여러번의 시도가 가능함
- `JobInstance`와 1:N 관계

![스크린샷 2022-04-06 오후 5 22 44](https://user-images.githubusercontent.com/60773356/162413924-79e1ab18-2734-4aca-ba51-aeb8ff259253.png)


---

## Step
- `Batch Job`을 구성하는 독립적인 하나의 단계, 실제 배치를 처리를 정의하고 컨트롤하는데 필요한 모든 정보를 가지고 있는 객체
- 입출력과 처리와 관련된 복잡한 비즈니스 로직 등 모든 설정을 담고 있음
- 배치 작업을 어떻게 구성하고 실행할 것인지 `Job`의 세부작업을 명세한 객체
- 기본 구현체
	- `TaskletStep`
		- 가장 기본이 되는 클래스, `Tasklet`타입의 구현체 제어
	- `PartitionStep`
		- 멀티 스레드 방식으로 `step`을 분리해 실행
	- `JobStep`
		- `Step`내에서 `Job`을 실행
	- `FlowStep`
		- `Step`내에서 `Flow`를 실행

![스크린샷 2022-04-07 오후 10 17 41](https://user-images.githubusercontent.com/60773356/162413952-90e2fbda-238c-4d0e-89eb-5e181b4f64a3.png)



---

## StepExecution
- `Step`에 대한 한번의 시도를 의미하는 객체, `Step` 실행 중 발생한 정보를 저장하는 객체
- 매번 시도될 때마다 생성되고 각 `Step`별로 생성
- `Job`이 재시작할 때 이미 성공한 `Step`은 실행되지 않고 실패한 것만 실행
- `Step`의 `StepExecution`이 모두 성공해야 `JobExecution`이 정상 완료
- `Step`의 `Execution`중 하나라도 실패하면 `JobExecution`은 실패
- `JobExecution`과 1:N 관계

![스크린샷 2022-04-07 오후 11 02 51](https://user-images.githubusercontent.com/60773356/162413981-165c5744-5948-44ee-97f6-438762e9bcb5.png)


---

## StepContribution
- `Chunk` 프로세스의 변경 사항을 버퍼링한 후 `StepExecution` 상태를 업데이트하는 도메인 객체
- `Chunk Commit`직전에 `StepExecution`의 `apply` 메서드를 호출해 상태 업데이트
- `ExitStatus`의 기본 종료 코드외 커스텀하게 종료 코드 생성가능

![스크린샷 2022-04-07 오후 11 15 02](https://user-images.githubusercontent.com/60773356/162413996-088b8cab-df90-4019-8379-b658588ff469.png)

![스크린샷 2022-04-07 오후 11 19 08](https://user-images.githubusercontent.com/60773356/162414009-af2e1c4c-58fa-4021-ba37-3068e9771af2.png)

- `Chunk`기반의 프로세스를 처리하는 시점에 생성된다.
- 생성 -> `execute()` -> `apply()`


---

## ExecutionContext
- `Framework`에서 유지 및 관리하는 `key-value`로 구성된 `Collection`
- `StepExecution`, `JobExecution` 객체의 상태를 저장하는 공유 객체
- 직렬화된 값으로 데이터베이스에 저장
- 공유 범위
	- `Step` 범위: 각 `Step`의 `StepExecution`에 저장되고 `Step`간 공유 x
	- `Job` 범위: 각 `Job`의 `JobExecution`에 저장되며 `Job`간 서로 공유 x, 같은 `Job` 내부의 `Step`간 서로 공유 o

![스크린샷 2022-04-08 오후 3 48 05](https://user-images.githubusercontent.com/60773356/162414026-87f8771d-0da3-4d32-a808-78de479ddb7b.png)

---

## JobRepository
- 배치 작업 중의 정보를 저장하는 저장소
- `Job`의 시작시간, 종료시간, 실행 횟수, 결과 등 모든 `meta data`를 저장
	- `JobLauncher`, `Job`, `Step` 구현체 내부에서 `CRUD` 기능을 처리

![스크린샷 2022-04-10 오후 4 58 42](https://user-images.githubusercontent.com/60773356/162613464-ab4ff35b-27e6-43e2-adb9-2fcf487e7e77.png)


### JobRepository 설정
- `@EnableBatchProcessing`만 선언하면 `JobRepository`가 스프링 빈으로 자동 등록
- `BatchConfigurer`,를 구현하거나  `BasicBatchConfigurer`를 상속해 설정을 커스터마이징 가능
	- `JobRepositoryFactoryBean`: `JDBC` 방식
	- `MapJobRepositoryFactoryBean`: `In Memory` 방식


### JobExecutionListener
![스크린샷 2022-04-10 오후 6 29 52](https://user-images.githubusercontent.com/60773356/162613469-429abf63-0698-4633-96b2-2ac08bbc5242.png)

- `JobExecutionListener`를 구현한 클래스를 만들어 `Listener`를 만든다.
- `beforeJob`과 `afterJob` 메서드를 오버라이딩하여 필요한 작업을 수행할 수 있음
- 파라미터로 주어지는 `JobExecution`을 통해 가져온 `Job`의 이름과 실행시점에 주어진 `JobParameter`를 통해 `JobRepository`에서 마지막으로 수행된 `JobExecution`을 꺼낼 수 있음

![스크린샷 2022-04-10 오후 6 33 09](https://user-images.githubusercontent.com/60773356/162613471-c2e2fdfa-eb18-431b-8f32-f5a83d29c4a2.png)

- 위에서 만든 `Listener`를 `Job`에 등록


![스크린샷 2022-04-10 오후 6 34 42](https://user-images.githubusercontent.com/60773356/162613480-1574fac8-8513-475f-9432-a3f2af7b459f.png)

- `ExecutionListener`를 통해 배치 정보들이 로그로 출력됨


![스크린샷 2022-04-10 오후 6 39 07](https://user-images.githubusercontent.com/60773356/162613482-a0806f5f-c073-4963-8feb-d33faebbf677.png)

- `JobRepository`에 제공되는 메서드가 있으니 상황에 맞게 사용하자


### BasicBatchConfigurer
![스크린샷 2022-04-10 오후 6 49 16](https://user-images.githubusercontent.com/60773356/162613497-aec098a2-710f-4ae7-b51e-689811b05de2.png)

- `BasicBatchConfigurer`를 상속하여 커스텀하게 설정할 수 있음
- 격리레벨, 테이블 prefix, 트랜잭션 매니저 등등
- 이 때, 테이블 prefix를 변경하려면 자동으로 생성된 테이블 `BATCH_XXX`의 prefix를 모두 원하는 prefix로 미리 변경해주거나 별도로 생성해주어야 함
	- 그렇지 않으면 해당 테이블이 존재하지 않는다는 오류 발생


![스크린샷 2022-04-10 오후 6 46 04](https://user-images.githubusercontent.com/60773356/162613490-4f55feb4-7fcc-42ea-b2f7-c93ad139e214.png)

- `JDBC` 방식으로 동작하기 때문에 주어지는 `setXXX` 메서드를 통해 값을 원하는대로 설정이 가능
