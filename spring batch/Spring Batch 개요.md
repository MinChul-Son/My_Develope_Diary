# Spring Batch 개요
## 1. 스프링 배치 탄생 배경
- 자바 기반 표준 배치 기술 부재
	- 배치 처리에서 요구하는 재사용 가능한 자바 기반 배치 아키텍처의 필요성
- `Accenture`는 이전에 소유했던 배치 처리 아키텍처 프레임워크를 `Spring Batch`에 기증


## 2. 배치 핵심 패턴
- Read: 데이터베이스, 파일, 큐에서 다량의 데이터 조회
- Process: 특정 방법으로 데이터 가공
- Write: 데이터를 수정된 양식으로 다시 저장


## 3. 배치 시나리오
- 배치 프로세스 주기적으로 커밋
- 동시 다발적인 `Job`의 배치 처리, 대용량 병렬 처리
- 실패 후 수동 또는 스케줄링에 의한 `Retry`
- 의존 관계가 있는 `step` 여러개를 순차적으로 처리
- 조건적 `Flow`구성을 통한 체계적으로 유연한 배치 모델 구성
- 반복, 재시도, `Skip` 처리


---

## 🌱 아키텍처

<img width="367" alt="스크린샷 2022-03-24 오후 5 31 07" src="https://user-images.githubusercontent.com/60773356/159940746-c73554dd-e8da-4667-a555-f8946a28fb41.png">

- Application
	- 스프링 배치 프레임워크를 통해 개발자가 만든 모든 배치 `Job`과 커스텀 코드를 포함
	- 개발자는 로직 구현에만 집중하고 공통 기반 기술은 프레임워크가 담당(IoC)
- Batch Core
	- `Job`을 실행, 모니터링, 관리를하는 `API`로 구성
	- `JobLauncher`, `Job`, `Step`, `Flow`등이 속함
- Batch Infrastructure
	- `Application`, `Core`모두 공통 `Infrastructure`위에서 동작
	- `Job` 실행 흐름과 처리를 위한 툴 제공
	- `Reader`, `Processor Writer`, `Skip`, `Retry`등이 속함

![스크린샷 2022-03-24 오후 5 38 17](https://user-images.githubusercontent.com/60773356/159940786-a9f8268d-d7ac-477a-abad-e84ecaa4ad6d.png)
