# ItemReader 활용

## FlatFileItemReader
- 2차원 데이터로 표현된 파일을 처리하는 `ItemReader`
- 일반적으로 고정 위치로 정의된 데이터 필드나 특수 문자에 의해 구별된 행을 읽음
- `Resource`와 `LineMapper` 두가지가 필요함
	- `Resource`: 읽어야할 리소스
	- `LineMapper`: String을 Objecti로 변환

<br>


### Resource
- FileSystemResource
- ClassPathResource

<br>

### LineMapper
- 라인 한줄을 `Object`로 변환해 `FlatFileItemReader`로 반환
- 문자열을 토큰화해 객체로 매핑하는 과정이 필요
- `LineTokenizer`, `FieldSetMapper`를 사용해 처리
	- `LineTokenizer`: 라인을 `FieldSet`으로 변환
	- `FieldSetMapper`: `FieldSet` 객체를 받아 원하는 객체로 매핑
- 라인을 필드로 구분해 만든 배열 토큰을 전달하면 토큰을 `FieldSet`을 통해 참조
	- `new String[]{"hello", "world"}`를 `FieldSet`에 전달하면 참조가능
	- 추상화된 `Spring Batch`의 인터페이스
	- `JDBC`의 `ResultSet`과 유사

<br>

### DelimitedLineTokenizer
- 한 라인을 구분자 기준으로 나누어 토큰화하는 방식을 사용
- 구분자의 기본값은 콤마
- `AbstractLineTokenizer` 추상클래스를 상속한 클래스
	- `AbstractLineTokenizer`는 `LineTokenizer` 인터페이스를 구현

<br>

### FixedLengthTokenizer
- 한 라인을 설정한 고정길이 기준으로 나누어 토큰화하는 방식을 사용
- 범위를 문자열 형식으로 지정
	- `"1~4"`

<br>

### Exception Handling
- 라인을 읽거나 토큰화할 때 발생하는 `Parsing` 예외를 처리할 수 있는 예외 계층 제공
- 토큰화 검증을 적용하지 않도록 설정하면 `Parsing` 예외가 발생하지 않도록 할 수 있음
	- `strict` 옵션
- 예외 종류
	- `ItemReaderException`
		- `FlatFileParseException`: 파일을 읽어들이는 동안 발생한 예외
	- `FlatFileFormatException`: 토큰화하는 중 발생하는 예외
		- `IncorrectTokenCountException`: 컬럼 개수와 실제 토큰화한 컬럼수가 다를 때 발생하는 예외
		- `IncorrectLineLengthException`: 토큰화할 때 라인 전체 길이와 컬럼 길이의 총합이 일치하지 않을 때 발생하는 예외

<br>

---

<br>

## JsonItemReader
- `Json` 데이터의 `Parsing` 및 `Binding`을 `JsonObjectReader` 구현처에 위임해 처리하는 `ItemReader`
	- `JacksonJsonObjectReader`, `GsonJsonObjectReader` 구현체 제공

<br>

---

<br>

## DB

`Spring Batch`는 실시간 처리가 어려운 대용량 데이터 처리를 위한 두가지 방안을 제시한다. 
- `Cursor`
- `Paging`

<br>

### Cursor Based 처리
- `JDBC ResultSet`의 기본 메커니즘 사용
- 현재 행에 커서를 유지 -> 다음 데이터를 호출하면 다음 행으로 커서를 이동하며 데이터를 반환(`Streaming` 방식의 `I/O`)
- `ResultSet`이 `Open`될 때마다 `next()`메서드가 호출되어 데이터가 반환되며 객체로의 매핑이 일어남
- 커넥션이 연결되면 처리가 완료될 때까지 데이터를 읽기 때문에(커넥션을 유지) `DB`와 `Socket`의 `timeout`을 충분히 큰 값으로 설정해야함
- 모든 결과를 메모리에 할당하므로 메모리 사용량이 많아지는 단점 존재
- 커넥션 유지 시간과 메모리 공간이 충분한 대용량 데이터 처리에 적합

<br>

### Paging Based 처리
- 페이징 단위로 데이터를 조회하는 방식(`Page Size`)
- 페이징 단위로 커넥션을 맺고 끊기 떄문에 `timeout`은 거의 발생하지 않음
- `Offset`과 `Limit`를 사용
- 페이징 단위로 메모리에 할당하므로 메모리 사용량이 적어지는 장점 존재
- 커넥션 유지 시간과 메모리 공간을 효율적으로 사용해야하는 대용량 데이터 처리에 적합

<br>

### JdbcCursorItemReader
- `Cursor`기반의 `JDBC`구현체, `ResultSet`과 함께 사용되고 `DataSource`에서 커넥션을 얻어옴
- `Thread-Safe`하지 않기 때문에 동시성 이슈를 고려해야함(멀티 스레드에서)
	- `read()`메서드를 동시에 호출해서 가져올 수 있기 때문
- `fetchSize`는 `chunkSize`와 동일하게 하는 것이 좋음
	- `chunkSize`만큼 커밋이 되기 때문
