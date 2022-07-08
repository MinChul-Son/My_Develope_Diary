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
