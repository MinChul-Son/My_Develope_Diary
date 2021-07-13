# Web Server vs WAS
* 10분 테코톡- "희봉"님의 영상을 보고 작성된 글입니다.
* [영상링크](https://www.youtube.com/watch?v=NyhbNtOq0Bc&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)

## Web Server
### Web이 뭐야?
* 인터넷을 기반으로 정보를 공유, 검색할 수 있게 하는 서비스
* URL(주소), HTTP(통신 규칙), HTML(내용)
### 그럼 Server는?
* 클라이언트에게 네트워크를 통해 정보나 서비스를 제공하는 컴퓨터 시스템

### Web + Server
* 인터넷을 기반으로 클라이언트에게 웹 서비스를 제공하는 컴퓨터
* 웹서버에게 주소(url)를 가지고 통신규칙(HTTP)에 맞게 요청하면, 내용(HTML)을 응답받음.
* 클라이언트의 요청을 기다리고 웹 요청에 대한 데이터를 만들어 응답(데이터는 웹에서 처리가능한 HTML, CSS, 이미지 등 정적인 데이터 한정)
* 하지만 정적인 데이터만의 한계가 발생 ==> 동적으로 처리할 수는 없을까?

### 예시
![image](https://user-images.githubusercontent.com/60773356/125455025-3dcef2f9-63bb-42e8-b117-7e0db513a5ee.png)
![image](https://user-images.githubusercontent.com/60773356/125455080-390a52eb-fb42-467e-807b-c613384205b0.png)


--------------------
## Web Application Server(WAS)
* Web Application
  - 웹에서 실행되는 응용 프로그램
* 웹 애플리케이션과 서버 환경을 만들어 동작시키는 기능을 제공하는 소프트웨어 프레임워크(미들 웨어)
* 웹 애플리케이션을 실행시켜 필요한 기능을 수행하고 그 결과를 웹 서버에게 전달(동적인 페이지)
* php, jsp, asp와 같은 언어를 사용해 동적인 페이지를 생성할 수 있는 서버
* 프로그램 실행 환경과 DB 접속 기능 제공
* 비즈니스 로직 수행 가능
* 웹 서버 + 웹 컨테이너(- jsp, servlet을 실행시킬 수 있는 SW)
* 자바 계열에서는 WAS를 웹 애플리케이션 컨테이너라고 부름.
  - 웹 애플리케이션이 배포되는 공간

### 동작과정
1. 클라이언트가 WAS 내부의 Web Server로 요청을 보냄.
2. Web Server는 받은 요청이 동적 페이지 요청이라면 Container로 전송
3. Container는 Servlet 구동 환경을 제공
4. Servlet이 구동되어 동적인 컨텐츠가 생성
5. 생성된 동적인 컨텐츠를 Web Server로 전달
6. Web Server는 Container로부터 받은 동적인 컨텐츠를 클라이언트에게 전달

### 예시
![image](https://user-images.githubusercontent.com/60773356/125455154-30d917ea-479c-4597-80a2-5c5b8f424493.png)
![image](https://user-images.githubusercontent.com/60773356/125455193-e9403886-63ab-4f86-82bb-3d1d26d4fd55.png)
![image](https://user-images.githubusercontent.com/60773356/125455215-eb6d232f-f94a-457e-92d4-eb74cf4fe731.png)

----------------------
## 결론
* 웹 서버는 정적인 컨텐츠만 제공 가능
* WAS는 애플리케이션을 실행하고 연결된 DB에서 데이터를 가져와 가공하여 줄 수 있는 서버, 즉 동적인 컨텐츠 제공 가능
#### 차이점 : 상황에 따라 변하는 정보를 제공할 수 있는가?




