# javax에서 제공하는 @Valid, Spring이 제공하는 BindingResult

## @Valid
* springboot가 버전업을 하면서 web 의존성안에 있던 constraints packeage가 모듈로 빠졌다. 
* SpringBoot 2.3 이상에서는 따로 의존성을 추가해주어야한다.
* @Valid를 이용하면, service 단이 아닌 객체 안에서, 들어오는 값에 대해 검증을 할 수 있다.
* javax.validation.constraints 패키지를 보면 많은 어노테이션들이 존재한다. @Valid를 이용한 객체 검증 시 기본적으로 이 어노테이션을 이용
* [공식문서](https://docs.jboss.org/hibernate/beanvalidation/spec/2.0/api/)
  ```
  implementation 'org.springframework.boot:spring-boot-starter-validation'
  ```
* 코드 예시)
  - 컨트롤러
    ```java
    @Controller
    @Slf4j
    public class MemberController {

        @PostMapping("/members/new")
        public String create(@Valid MemberForm form) {
            log.info("회원 폼 객체 검증 성공!!");
            return "redirect:/";
        }
    }
    ```
  - 회원 폼 객체
    ```java
    @Getter @Setter
    public class MemberForm {
    
      @NotEmpty(message = "회원 이름은 필수 입니다.") // null, "" 문자는 불가능
      private String name;

      private String city;
      private String street;
      private String zipcode;
    }
    ```
#### 위와 같은 상황에서 이름을 비워놓은 채로 객체를 post방식으로 전달하면 400(Bad Request)에러가 발생한다. 
#### @NotNull, @NotEmail 등등 많은 검증 애노테이션을 제공한다.


-------------------------------------------------
----------------------------------------------
## BindingResult
#### Spring이 제공하는 BindingResult와 @Valid를 함께 사용하면 에러 처리를 더욱 깔끔하게 할 수 있다.
```java
@Controller
@Slf4j
public class MemberController {

  @PostMapping("/members/new")
  public String create(@Valid MemberForm form, BindingResult result) {
    if (result.hasErrors()) {
        log.info("에러 발생!!");
        return "redirect:/";
      }
    log.info("회원 폼 객체 검증 성공!!");
    return "redirect:/";
  }
}
```
* 위와 같이 @Valid 뒤에 BindingResult 객체를 사용하면 @Valid에서 오류가 발생하면 그 오류가 result에 담기게 된다.
* 이를 hasErrors()를 사용하여 에러가 존재하는지 유무를 판별할 수 있다.
* 좀더 응용하여 타임리프와 함께 사용한다면 나타내고자하는 바를 확실하게 나타낼 수 있다.
```html
<input type="text" th:field="*{name}" class="form-control" placeholder="이름을 입력하세요"
       th:class="${#fields.hasErrors('name')}? 'form-control fieldError' : 'form-control'">
        <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Incorrect date</p>
```
* 타임리프 문법을 사용하여 에러가 있을 때만 에러의 내용을 출력하게 할 수 있다.
![image](https://user-images.githubusercontent.com/60773356/119220694-b5e88d00-bb26-11eb-9f82-45c6bd177403.png)
- 이름을 비운 채로 요청을 전송했을 때 오류 메시지가 출력되었다.











