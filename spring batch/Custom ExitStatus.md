# Custom ExitStatus	
- `ExitStatus`에 존재하지 않는 `exitCode`를 새로 정의
- `StepExecutionListener`의 `afterStep()`메서드에서 커스텀 `exitCode`를 통해 새로운 `ExitStatus`를 반환할 수 있음
- `Step`실행 완료 시점에 커스텀한 `exitCode`로 수정할 수 있음



![스크린샷 2022-05-18 오후 11 27 10](https://user-images.githubusercontent.com/60773356/169696606-8af53a67-e56b-4b49-94a4-50e53a21e4fe.png)

- 이와 같은 코드에서 `step1`은 정상 종료되었고 `ExitStatus`만 `FAILED`로 변경해주었음
- DB에 저장된 값을 확인해보면 `JOB_EXECUTION`의 `STATUS`도 `FAILED`로 저장됨
- `JOB_EXECUTION`은 마지막 `STEP`의 성공, 실패 여부에 따라 값이 저장되는데 위의 코드에서 `step2`의 종료 코드가 `PASS`면 그 위의 `Trasition`을 수행하고 있음
- 이 때 지정된 코드가 아닐 경우에 `Spring Batch`가 `Step`이 실패되었다고 판단하기 때문


