# π± Step

## StepBuilderFactory
- `StepBuilder`λ₯Ό μμ±νλ ν©ν λ¦¬ ν΄λμ€

<br>

## StepBuilder
- `Step`μ κ΅¬μ±νλ μ€μ  μ‘°κ±΄μ λ°λΌ νμ λΉλ ν΄λμ€λ₯Ό μμ±νκ³  `Step`μμ±μ μμ
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
- `Step`μ κ΅¬νμ²΄, `Tasklet`μ μ€νμν€λ λλ©μΈ κ°μ²΄
- `RepeatTemplate`λ₯Ό μ¬μ©ν΄ `Tasklet`μ λ‘μ§μ νΈλμ­μ κ²½κ³ λ΄μμ λ°λ³΅ μ€ν
	- νΈλμ­μ μ²λ¦¬κ° λ³λλ‘ νμμμ, μ€νλ§ λ°°μΉ λ΄λΆμ μΌλ‘ μ²λ¦¬λμ΄μμ
- `Task`κΈ°λ°κ³Ό `Chunk` κΈ°λ°μΌλ‘ λλμ΄ μ€ν

<br>

### Task κΈ°λ°
- λ¨μΌ μμ κΈ°λ°μΌλ‘ μ²λ¦¬λλ κ²μ΄ λ ν¨μ¨μ μΈ κ²½μ°
- `Tasklet` κ΅¬νμ²΄λ₯Ό λ§λ€μ΄ μ¬μ©
- `Chunk` κΈ°λ°μ λΉν΄ λ λ³΅μ‘ν κ΅¬ν νμ

<br>

### Chunk κΈ°λ°
- λλ μ²λ¦¬λ₯Ό νλ κ²½μ° ν¨κ³Όμ 
- `ItemReader`, `ItemProcessor`, `ItemWriter`λ₯Ό μ¬μ©
-  `ChunkOrientedTasklet` κ΅¬νμ²΄κ° μ κ³΅λ¨

<br>

### API
- `tasklet()`: `Tasklet` ν΄λμ€ μ€μ 
	- `Chunk` or `Task`
- `startLimit()`: `Step`μ μ€ν νμ μ€μ , μ΄κ³Όμ μμΈ λ°μ
- `allowStartIfComplete()`: `Step`μ μ±κ³΅, μ€ν¨ μ¬λΆμ μκ΄μμ΄ ν­μ `Step`μ μ€ννκΈ° μν μ€μ 
- `listener()`: νΉμ  μμ μ μ½λ°±μ μν λ¦¬μ€λ μ€μ 
- `build()`: `TaskletStep`μ μμ±

<br>

### tasklet()
- `Tasklet` νμμ ν΄λμ€λ₯Ό μ€μ 
- μ΅λͺ ν΄λμ€ or κ΅¬νμ²΄λ₯Ό λ§λ€μ΄ μ¬μ©
- `TaskletStepBuilder`κ° λ°ν
- `Step`μ νλμ `Tasklet` μ€μ  κ°λ₯, λκ° μ΄μμ μ€μ νλ©΄ λ§μ§λ§ μ€μ  κ°μ²΄λ§ μ€νλ¨

<br>

### startLimit()
- `Step`μ μ€ν νμλ₯Ό μ‘°μ 
- `Step`λ§λ€ μ€μ  κ°λ₯
- μ΄κ³Όν΄μ μ€ννλ©΄ `StartLimitExceededException` λ°μ
- μ€ννλ©΄  `JobExecution`μ κ³μ μμ±λμ§λ§ `StepExecution`μ μ€μ λ κ° μ΄μμΌλ‘ μμ±λμ§ μμ

<br>

### allowStartIfComplete()
- μ¬μμ κ°λ₯ν `Job`μμ `Step`μ μ±κ³΅ μ¬λΆμ κ΄κ³μμ΄ ν­μ μ€ννκΈ° μν μ€μ 
- μ€νλ§λ€ μ ν¨μ±μ κ²μ¦νλ `Step`, μ¬μ  μμ `Step`μ μ¬μ©νλ©΄ μ μ©ν μ΅μ
- `true`λ‘ μ€μ λλ©΄ `COMPLETED` μνκ° λλλΌλ ν­μ μ€ν

<br>
<br>

---

## JobStep
- `Job`μ μνλ `Step` μ€ μΈλΆμ `Job`μ ν¬ν¨νκ³  μλ `Step`
	- μ¦, `Step`μμ λλ€λ₯Έ `Job`μ΄ μ‘΄μ¬
- λͺ¨λ  λ©νλ°μ΄ν°λ κΈ°λ³Έ `Job`κ³Ό μΈλΆ `Job`λ³λ‘ κ°κ° μ μ₯
- λ΄λΆμ `Job`μ΄ μ€ν¨νλ©΄ μΈλΆμ `Job`λ μ€ν¨
- `JobParamtersExtractor`λ `ExecutionContext`λ΄μ `key`λ₯Ό μ°Ύμμ€λ μ­ν 
	- `Spring Bean`μΌλ‘ λ±λ‘νμ§ μμλλ¨


