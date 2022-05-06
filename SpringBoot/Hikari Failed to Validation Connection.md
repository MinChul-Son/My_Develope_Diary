# Hikari Failed to Validation Connection

![image](https://user-images.githubusercontent.com/60773356/167066809-639f238a-e05f-465d-8688-7b8fb7fd6ca3.png)
 
서버 모니터링을 진행하다가 최근에 마주친 `Warn` 로그이다.
 
로그를 읽어보면 `Hikari Pool`의 커넥션을 `validation`하는데 실패하였고 이미 닫힌 커넥션을 조작하는 것은 불가하며, 커넥션의 `maxLifetime`을 짧게 설정하는 것을 고려해보라고 말하고 있다.
 
---
 
## Hikari Pool의 max-lifetime
먼저, `Hikari Pool`의 `max-lifetime`에 대해 이야기해보자.

커넥션을 생성하는 비용을 최적화하기 위해 커넥션 풀을 사용해 커넥션을 관리하는데 `SpringBoot 2.0`이후에서는 `HikariCP`를 기본옵션으로 사용하고 있다.

여기서 `max-lifetime`이란 말그대로 `Hikari Pool`에서 커넥션이 살아 있을 수 있는 최대 시간을 의미한다.(최소 30초 이상으로 설정해야함)

`default`는 30분이고 `0`을 주면 무한으로 커넥션이 살아있게 된다.

`HikariCP`는 `Connection`이 생성될 때 `Scheduled Event`를 걸어 `maxLifetime`이후에 강제적으로 `Connection`을 종료한다.


## MySQL의 wait_timeout
해당 로그가 발생한 원인을 분석하기 위해 `MySQL`의 `wait_timeout`에 대해 알아보자.

`wait_timeout`도 옵션명으로 유추가 가능하듯이 아무런 동작없이 대기 중인 커넥션을 주어진 시간 이후에 끊어버리는 옵션이다.


---

## 원인
즉, 로그가 발생했던 이유는 `MySQL`의 `wait_timeout`으로 끊어진 커넥션에 `Hikari CP`가 `max-lifetime`마다 커넥션을 만지려고 했기 떄문에 발생했던 것이다.

나의 경우에는 `max-lifetime`은 설정하지 않았기 떄문에 디폴트로 30분이 설정되어 있었고,

`MySQL`의 `wait_timeout`은 300초로 설정되어 있었다.
> show global variables like 'wait_timeout'; # `wait_timeout`값 확인하는 쿼리

## 해결법
`max-lifetime`을 `wait_timeout`보다 몇초정도 짧게 설정하면 해당 경고 로그는 사라진다.

## 참고 이슈
https://github.com/brettwooldridge/HikariCP/issues/1483
