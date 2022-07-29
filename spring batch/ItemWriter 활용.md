## FlatFileItemWriter
- 2차원 데이터로 표현된 유형의 파일을 처리하는 `ItemWriter`
- `Resource`와 `LineAggregator` 두가지가 필요
	- `LineAggregator`: `Object`를 `String`으로 변환하는 역할
		- 내부적으로 `FieldExtractor`를 사용해 처리
		- `PassThroughLineAggregator`, `DelimitedLineAggregator`, `FormatterLineAggregator`가 구현체로 제공됨
- `FieldExtractor`: 전달 받은 `Item`의 필드를 배열로 만들고 그 배열을 합쳐 문자열을 만드는 방식으로 동작하게 제공되는 인터페이스
	- `BeanWrapperFieldExtractor`, `PassThroughFieldExtractor`가 구현체로 제공됨

<br>

### DelimitedLineAggregator
- 객체 필드 사이에 구분자를 삽입한 문자열로 반환

<br>

### FormatterLineAggregator
- 객체의 필드를 사용자가 설정한 `Formatter` 구문을 사용해 문자열로 변환

---

<br>

## JsonFileItemWriter
- 객체를 받아 `Json String`으로 변환


---

<br>

## JdbcBatchItemWriter
- `datasource`를 지정, `sql`속성에 쿼리 설정해야함
- `JDBC`의 `Batch`기능을 사용해 `bulk`방식의 처리
- 일괄처리이므로 성능상 이점이 존재함


---

<br>

## JpaItemWriter
- `Jpa Entity` 기반으로 데이터를 처리
	- `EntityManagerFactory`를 주입받아 사용
- `Entity`를 `chunk` 크기만큼 `insert`, `merge`한다음 `flush`
- `ItemReader`, `ItemProcessor`로부터 아이템을 전달받을 때는 `Entity` 타입으로 받아야함


---

<br>

## ItemWriterAdapter
- `Job`안에서 이미 있는 서비스를 `ItemWriter`에서 사용하고자할 때 위임 역할을 함

