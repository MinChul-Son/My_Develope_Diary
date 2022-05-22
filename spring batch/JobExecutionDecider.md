# JobExecutionDecider
- `ExitStatus`를 조작하거나 `StepExecutionListener`를 등록할 필요없이 `Transition`처리를 위한 클래스
- `Step`과 `Transition` 역할을 분리해서 설정 가능
- `JobExecutionDecider`의 `FlowExecutionStatus` 상태값을 새롭게 설정해 반환

 


![스크린샷 2022-05-19 오후 2 46 51](https://user-images.githubusercontent.com/60773356/169696645-f9d02c38-5af4-4945-b98e-9824a1dc8822.png)
- `JobExecutionDecider`를 파라미터로 넘겨 해당 `Decider`의 상태에 따라 다른 `Step`을 실행하도록 정의

![스크린샷 2022-05-19 오후 2 47 00](https://user-images.githubusercontent.com/60773356/169696648-4cff471b-a39e-4274-acf1-8cbf4a32b3f7.png)


