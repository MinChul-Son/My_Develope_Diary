# 도커 컨테이너
![image](https://user-images.githubusercontent.com/60773356/130023701-598e4d87-7177-4eee-b651-92580cdc5be4.png)

`vmware`과 같은 가상머신에서 별도의 `OS`를 설치하는 것은 결국 하나의 컴퓨터 자원을 공유하는 비용 문제가 발생하게 된다. 또 각각의 환경에 따라 계속해서 가상환경을 세팅해줘야한다.

`container`는 개별 SW 실행에 필요한 실행환경을 독립적으로 운용할 수 있도록 격리된 환경을 제공하는 기술을 뜻한다.
이는 운영체제 커널을 공유하난 작고 가벼운 실행 환경으로 애플리케이션을 분리시킬 수 있어 가상 머신보다 훨씬 적은 리소스와 속도가 빠르다는 장점이 존재한다.
![image](https://user-images.githubusercontent.com/60773356/130024201-79ad9455-0251-46d7-a5b7-ea5903f8ac78.png)

`image`를 통해 관련 라이브러리 및 종속 항목과 함께 패키지로 묶어 하나의 컨테이너를 만들어내고 이는 별도의 격리된 환경이 된다.

각 `image`, 예를 들어 `mysql` / `python` / `node.js` 를 별도의 컨테이너로 만들면 이는 서로 데이터를 주고 받을 수 없다. 이를 위해 `docker compose`가 사용된다.
일반적으로 `docker-compose.yml`를 통해 복수 개의 컨테이너를 실행시키는 도커 애플리케이션의 서비스를 구성한다. 각 컨테이너들이 서로 데이터를 주고 받으면서 우리가 흔히 아는 `MSA`가 되는 것이다.


### 공식 문서
* [docker container](https://www.docker.com/resources/what-container)
