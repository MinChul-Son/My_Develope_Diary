# Optioanl 객체에 접근하기
- [Optional 공식문서](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)

## Optioanl이란?
- A container object which may or may not contain a non-null value.
- Optional이란 자바8부터 추가된 Wrapper Class이다. 
- 쉽게 말해 null을 담을 수 있다. 
- 모든 타입의 참조 변수를 저장할 수 있으므로 만약 null 값이 들어올 가능성이 있다면 Optional로 감써서 사용하는 것이 좋다.
- 가장 무서운 에러인 **NullPointerException**을 회피할 수 있다.

### Optional 객체에 접근하기
- Optional 객체에 get으로 바로 접근하는 것은 좋지않다라는 말을 들어본적이 있을 것이다. 그 이유가 무엇일까?
- 그 이유는 만약 Optional에 null 값이 담겨있을 때 get으로 접근하게되면 NoSuchElementException 예외가 발생하게 된다. 
- 따라서 null 값을 체크한 후에 get으로 접근하거나, 제공되는 null 처리 메서드를 사용하도록 한다.
    - orElse() 메소드 : 저장된 값이 존재하면 그 값을 반환하고, 값이 존재하지 않으면 인수로 전달된 값을 반환함.
    - orElseGet() 메소드 : 저장된 값이 존재하면 그 값을 반환하고, 값이 존재하지 않으면 인수로 전달된 람다 표현식의 결괏값을 반환함.
    - orElseThrow() 메소드 : 저장된 값이 존재하면 그 값을 반환하고, 값이 존재하지 않으면 인수로 전달된 예외를 발생시킴.
