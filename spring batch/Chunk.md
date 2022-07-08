# Chunk
- 여러 개의 아이템을 묶은 한 덩어리, 블록을 의미
- 한번에 하나씩 아이템을 입력받아 `Chunk`단위로 만든 후 트랜잭션을 처리
	- `Chunk`단위의 `Commit`과 `Rollback`이 이뤄짐
- 일반적으로 대용량 데이터를 한번에 처리하는 것이 아닌 `Chunk`단위로 쪼개 더 이상 처리할 데이터가 없을 때까지 반복해서 입출력하는데 사용함
- `ItemWriter`에는 하나씩이 아닌 리스트 형태의 `items`를 넘김

<br>

## ChunkOrientedTasklet
- 스프링 배치에서 제공하는 `Tasklet`구현체 중 `Chunk`지향 프로세싱을 담당하는 도메인 객체
- `ItemReader`, `ItemProcessor`, `ItemWriter`를 사용해 `Chunk`기반의 데이터 입출력 처리를 담당
- `TaskletStep`에 의해 반복 실행, `ChunkOrientedTasklet`이 실행될 때마다 새로운 트랜잭션이 생성됨
- 예외가 발생하면 해당 `Chunk`는 롤백됨
- 내부적으로 `ItemReader`를 핸들링하는 `ChunkProvider`, `ItemProcesser`와 `ItemWriter`를 핸들링하는 `ChunkProcessor`타입의 구현체를 가지고 있음

<br>

## ChunkProvider
- `ItemReader`를 사용해 아이템을 `Chunk Size`만큼 읽어 `Chunk`단위로 만들어 제공하는 도메인 객체
- `Chunk<I>`를 만들고 내부 반복문을 통해 `ItemReader.read()`를 호출
- 기본 구현체 -> `SimpleChunkProvider`, `FaultTolerantChunkProvider`


## ChunkProcessor
- `ItemProcessor`를 사용해 `Item`을 변형, 가공, 필터링하고 `ItemWriter`를 사용해 `Chunk`데이터를 저장 및 출력
- `Chunk<O>`를 만들어 `Chunk<I>`를 하나씩 처리후 저장
- `ChunkProcessor`가 호출될때마다 새로운 `Chunk` 생성
- `ItemProcessor`는 선택사항 -> 객체가 없으면 아이템 그대로 `Chunk<O>`에 저장
- 처리가 완료되면 `Chunk<O>`의 `List<Item>`을 `ItemWriter`에게 전달
- `ItemWriter`의 처리가 완료되면 `Chunk` 트랜잭션 종료, 새로운 `ChunkOrientedTasklet` 실행
- 기본 구현체 -> `SimpleChunkProcessor`, `FaultTolerantChunkProcessor`

<br>

## ItemReader
- 다양한 입력으로부터 데이터를 읽어 제공하는 인터페이스
	- `Flat`, `XML`, `JSON`, `DB`,  `MQ` 등
- `ChunkOrientedTasklet` 실행 시 필수 요소
- 다수의 구현체가 `ItemReader`와 `ItemStream`을 다중 구현하고 있음

<br>
 
## ItemWriter
- `Chunk`단위로 데이터를 받아 일괄 출력을 위한 인터페이스
- 아이템 리스트를 전달받음
- `ChunkOrientedTasklet`실행 시 필수 요소
- 다수의 구현체가 `ItemWriter`와 `ItemStream`을 다중 구현하고 있음

<br>

## ItemProcessor
- 데이터를 출력하기 전에 가공, 변형, 필터링하는 역할
- 비즈니스 로직을 구현할 수 있음
	- 특정 타입으로 변환
	- 필터링
- `ChunkOrientedTasklet` 실행 시 선택 요소
- `ItemStream`을 구현하지 않음
- 대부분 커스터마이징을 하여 사용하므로 기본 구현체가 적음

<br>

## ItemStream
- `ItemReader`와 `ItemWriter`의 처리 과정 중 상태를 저장하고 오류가 발생하면 해당 상태를 참조하여 실패한 지점에서 재시작을 지원함
- `ItemReader`, `ItemWriter`는 `ItemStream`을 구현해야함
- `ExecutionContext`와 연계해서 사용가능
- 리소스를 열고 닫는 `open()`, `close()` 메서드와 상태를 저장하는 `update()`메서드가 존재
	- `chunk size`만큼 읽어오고 `update()`호출
