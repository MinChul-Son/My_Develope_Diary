# 도커 이미지
`Docker Hub`에 존재하는 파일을 `image`라고 부른다. `docker pull`

`image`를 실행하는 것을 `container`라고 부른다. `docker run`. `image`는 여러 개의 `container`를 가질 수 있다.

`image`는 `container` 실행에 필요한 파일과 설정 값등을 포함하고 있는 것으로 불변(Immutable)이다. `container`의 상태가 바뀌거나 삭제되어도 `image`는 변하지 않고 그대로 남아있다.

만약 내가 `Docker Hub`에서 `Python` 공식 이미지를 내려받을 때 작성한 커스텀 `Dockerfile`을 통해 필요한 라이브러리, 파이썬의 버전 등을 설정할 수 있다.
* ex) `python 3.7`버전, `flask`을 다운받은 상태의 `image`로 만들 수 있음

따라서, 이 `Dockerfile`을 다른 곳에서도 사용할 수 있어 **다른 컴퓨터에서 동일한 환경을 세팅할 수 있다!!**

![image](https://user-images.githubusercontent.com/60773356/130023302-3a47f7ae-47ca-4248-92b7-2913d6f1cd90.png)


### 공식 문서
* [Docker overview](https://docs.docker.com/get-started/overview/)
* [docker images](https://docs.docker.com/engine/reference/commandline/images/)














