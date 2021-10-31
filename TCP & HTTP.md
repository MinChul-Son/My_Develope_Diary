# TCP 와 HTTP
`HTTP 완벽 가이드`라는 책을 읽으며 프로젝트를 함께 진행하는 팀원들과 스터디를 진행하고 있다.

진행을 하며 팀원분이 직접 패킷을 까보며 진행하면 더 좋을 것 같다는 방향성을 제시해주셨고, 최종적으로 `Cisco Packet Tracer`를 사용하기로 하였다.

`4장. 커넥션 관리`라는 주제에 대한 발표를 맡게되어 `Cisco Packet Tracer`를 사용해 직접 흐름을 보며 패킷을 까보았던 경험이 정말 좋은 경험이었기에 글로 남기려한다.

[책 내용 정리한 발표자료](https://github.com/MinChul-Son/Book-for-Developer/blob/main/HTTP%20%EC%99%84%EB%B2%BD%20%EA%B0%80%EC%9D%B4%EB%93%9C/4%EC%9E%A5.%20%EC%BB%A4%EB%84%A5%EC%85%98%20%EA%B4%80%EB%A6%AC.md)

해당 글에서는 `Cisco Packet Tracer`를 통한 분석만을 작성할 예정이므로 자세한 내용은 위 글을 참조하자.

<br>

---------------------------------------

우리가 `www.naver.com`이라는 `url`을 브라우저에 입력할 때 어떤 일이 일어나는 지에 대해 생각해본적이 있는가?
- `www.naver.com`이라는 도메인 주소를 사용하는 서버의 `ip`주소를 알아내기 위해 `DNS` 서버에게 물어본다.
- 1번의 과정으로 알아낸 서버의 주소와 포트를 지정하지 않았다면 `well known` 포트를 통해 서버에 접근한다. ==> `http는 80, https는 443` 
  - 즉, `https://www.naver.com`와 `https://www.naver.com:443`은 같다.
- 클라이언트와 서버가 `TCP Connection`을 맺는다.
- HTTP 통신을 한다.(클라이언트의 요청, 서버의 응답)
- TCP 커넥션을 종료한다.(지속 커넥션이라면 종료 안함)

매우 간단해 보이는 `HTTP 트랜잭션` 한 과정에서 위와 같은 매우 많은 통신이 일어난다.

각설하고, `Packet Tracer`를 통해 직접 살펴보자.

아래는 웹 브라우저를 통해 `ark-inflearn.com`이라는 도메인 주소로 접속했을 때의 전체 흐름이다.(사전에 DNS서버에 서버 ip 주소를 `ark-inflearn.com`이라는 도메인으로 등록해둠)

![ezgif com-gif-maker (1)](https://user-images.githubusercontent.com/60773356/139578650-7e2ec2d6-18e9-4060-b95c-99c25d7285ca.gif)

위에서 언급된 과정과 동일한 흐름으로 진행되는 것을 확인할 수 있다.
- ip 주소 추출
- 추출된 ip주소와 포트 번호를 통해 `TCP Connection` 생성 ==> `3-Way Handshake`
- 클라이언트에서 서버로 `HTTP Request` 전송
- 서버에서 클라이언트로 `HTTP Response` 전송
- `TCP Connection` 종료 ==> `4-Way Handshake`

<br>

각 단계에 따른 패킷을 직접 분석해보자.

네트워크는 `Client`, `Switch`, `DHCP Server`, `DNS Server`, `Web Server`로 구성되어있다.
- `Client`의 ip주소 : `192.168.100.1`
- `DNS Server`의 ip주소 : `192.168.100.254`
- `Web Server`의 ip주소 : `192.168.100.253`

## 1. DNS
**클라이언트가 브라우저에 입력한 도메인으로부터 서버의 `ip`를 추출해내는 과정**이다.

`DNS Server`에 등록된 도메인인지? 등록되었다면 `ip`주소가 무엇인지를 물어본다.

![image](https://user-images.githubusercontent.com/60773356/139579088-c1afbb70-f802-4cad-9be7-d64d0166c706.png)
- `UDP` 프로토콜을 사용하고, `Source`(발신지)는 `Client` 주소이고, `Destination`(수신지)는 `DNS Server`인 것을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/60773356/139579145-92f09c8f-d7ff-4e7d-83d4-d9b711163229.png)
- 패킷 내부를 보면 `DNS Query`를 통해 해당 호스트명을 사용하는 서버의 `ip`주소를 질의하고 있다.

<br>

이것이 `Switch`를 통해 `DNS Server`에 전달된다. 그 후, `DNS Server`는 해당 호스트명에 해당하는 서버의 `ip`주소를 담아 다시 `Client`에게 전송한다.

![image](https://user-images.githubusercontent.com/60773356/139579267-74f4c501-d009-4302-83fe-c77c13ac3b4a.png)
- In Layers : 요청
- Out Layers : 응답

`DNS Server`의 응답을 살펴보자.

위에서 확인할 수 있듯이 `Source`는 `DNS Server`, `Destination`은 `Client`이다.

![image](https://user-images.githubusercontent.com/60773356/139579378-34aa4360-e8f1-4b68-b817-9fade505e657.png)
- `DNS Query`에 대한 `DNS Answer`를 확인하면 `ark-inflearn.com`이라는 도메인을 사용하는 서버의 `ip`주소는 `192.168.100.253`이라고 응답하고 있다.
- 이는 `Client`가 접근하길 원하는 서버의 `ip`주소이다.

#### 이제 클라이언트는 서버의 `ip`주소와 포트번호를 알아냈다.

<br>

## 2. TCP Connection(3-Way HandShake)
요청을 전송할 목적지를 알아냈으므로 이제는 요청을 전달할 경로를 만들어야한다.

이 경로는 바로 `TCP Connection`이다.(세부 설명은 여기서 다루지 않는다.)

`TCP Connection`을 맺기 위해서는 `3-Way Handshake`가 일어나는데 아래에서 살펴볼 과정이 바로 그것이다.
- 클라이언트에서 서버로 `SYN` 전달
- 서버에서 클라이언트로 `ACK` + `SYN` 전달
- 클라이언트에서 서버로 `ACK` 전달 

<br>
<br>


#### 클라이언트가 서버와 `TCP Connection`을 맺고 싶다는 요청인 `SYN`패킷을 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139579553-ed1feca4-9e5f-43a8-95af-e413726a2a17.png)
> ![image](https://user-images.githubusercontent.com/60773356/139579617-e2066b2f-3bb3-4860-b0dd-1f96530cf4d7.png)

#### 서버가 클라이언트에게 `TCP Connection`을 맺고 싶다는 요청인 `SYN` 패킷과, 클라이언트의 `SYN`을 잘 받았다는 `ACK`패킷을 함께 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139579729-eed9bdcf-1aea-4466-b823-e3f88e17b081.png)

#### 클라이언트가 서버에게 `TCP Connection` 요청이 받아들여졌다는 `ACK`패킷을 전송한다.
![image](https://user-images.githubusercontent.com/60773356/139579794-c7690510-1aa8-4734-a7a8-80bd5d0bc80a.png)

<br>

#### 여기까지의 과정을 통해 현재 클라이언트와 서버는 `TCP Connection`을 생성한 상태이다. 

<br>

## 3. HTTP Transaction
`HTTP Request`와 `HTTP Response`로 이루어진 한 싸이클을 `HTTP Transaction`이라고 부른다.

앞선 2번을 통해 클라이언트와 서버는 `HTTP Transaction`을 위한 사전 작업인 `TCP Connection`을 생성하였다.

<br>

#### 클라이언트가 서버에게 HTTP Request를 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139580022-17e49d5a-d909-4329-8214-c5f34447eb0a.png)
> ![image](https://user-images.githubusercontent.com/60773356/139580071-25fa429a-2a64-484a-a95a-b38fdb2a7734.png)
- `Connection: close`를 통해 지속 커넥션을 사용하지 않고 `HTTP Transaction` 이후 커넥션을 종료할 것임을 명시함

#### 서버가 클라이언트에게 HTTP Response를 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139580169-7f6c53f7-7f70-415d-9f40-93792de6d4b7.png)
> ![image](https://user-images.githubusercontent.com/60773356/139580180-f50ff3af-9466-462f-a581-ac02f51ca896.png)
- `Connection`을 종료될 것임을 알려주고, `Content-Length`를 통해 본문의 길이와 `Content-Type`을 통해 본문의 타입에 대한 정보를 알려준다.
이렇게 클라이언트에게 `HTTP Response`가 반환되게되고 클라이언트가 원하는 데이터가 브라우저를 통해 화면에 띄워진다.

![image](https://user-images.githubusercontent.com/60773356/139580264-d4e9df12-9f42-4e6a-8bf0-180a8f5ec68b.png)

<br>

#### HTTP Transaction 성공적으로 완료되었고, 이제 `Connection: close`로 인한 `TCP Connection` 종료가 진행된다.

<br>

## 4. TCP Close
`TCP Connection` 생성과는 달리 종료시에는 `4-Way Handshake`가 수행된다.

![image](https://user-images.githubusercontent.com/60773356/139580362-0630848c-521d-4518-98cb-7617a38a5643.png)
- 클라이언트가 `FIN` 패킷을 서버로 전송한다.
- 서버는 커넥션을 종료하고 확인 응답인 `ACK` 패킷을 클라이언트로 전송한다.
- 서버는 `FIN` 패킷을 클라이언트로 전송한다.
- 클라이언트는 커넥션을 종료하고 확인 응답인 `ACK` 패킷을 클라이언트로 전송한다.

<br>

#### 클라이언트가 `FIN` 패킷을 서버로 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139580423-8cc3d005-8ff0-42a0-add0-35014848525e.png)

#### 서버가 `ACK`패킷을 클라이언트에게 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139580450-ac3784af-497b-46cc-ad12-d62231a2c76e.png)

#### 서버가 `FIN` 패킷을 클라이언트에게 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139580478-d7a1abe5-535b-40f7-b07f-4e140dd22db9.png)

#### 클라이언트가 `ACK`패킷을 서버에게 전송한다.
> ![image](https://user-images.githubusercontent.com/60773356/139580508-b615d889-30b8-45ea-8294-d8410ed7b8e1.png)

<br>

-------------------------------------------------------------

우리가 무심코 사용하는 `HTTP` 통신을 위해서는 이렇게 수많은 과정이 숨어있다. 






