##  Springboot Web Service

### 단위 테스트를 하는 이유

* TDD와 Unit Test(단위 테스트)
  * TDD는 테스트가 주도하는 개발을 이야기 한다.
  * 테스트 코드를 먼저 작성하는 것부터 시작
  * 레드 그린 사이클
    * Red: 항상 실패하는 테스트를 먼저 작성하고
    * Green: 통과하는 프로덕션 코드를 작성하고
    * Refactor: 테스트가 통과하면 프로덕션 코드를 리팩토링 한다.
* 단위 테스트는 개발단계 초기에 문제를 발견하게 도와준다.
* 단위 테스트는 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인할 수 있다.
* 단위 테스트는 기능에 대한 불확실성을 감소시킬 수 있다.
* 단위 테스트는 시스템에 대한 실제 문서를 제공한다. 즉 단위 테스트 자체가 문서로 사용될 수 있다.



1. 코드를 작성하고
2. 프로그램(Tomcat)을 실행한 뒤
3. Postman과 같은 API 테스트 도구로 HTTP 요청하고
4. 요청 결과를 System.out.println()으로 눈으로 검증한다.
5. 결과가 다르면 다시 프로그램(Tomcat)을 중지하고 코드를 수정한다.





- Application.java

  

  ``` java
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  // 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정됨.
  @SpringBootApplication
  public class Application {
      public static void main(String[]args) {
  		SpringApplication.run(Application.class, args);
      }
  }
  ```



##### WAS (Web Application Server) : 웹 애플리케이션 서버

- 내장 WAS란 별도로 외부의 WAS를 두지 않고 애플리케이션을 실행할 때 내부에서 WAS를 실행하는 것을 이야기함. 이렇게 되면 항상 서버에 Tomcat을 설치할 필요가 없게 되고, 스프링 부트로 만들어진 Jar 파일(실행 가능한 Java 패키징 파일)로 실행하면 된다.
- 내장 WAS의 장점은 '언제 어디서나 같은 환경에서 스프링 부트를 배포'할 수 있다.



### Controller.java

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

// 1.
@RestController
public class Controller {
    //2.
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
    
}
```

1. @RestController
   * 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
   * 예전에는 @ResponseBody를 각 메소드마다 선언했던 것을 한번에 사용할 수 있게 해준다.
2. @GetMapping
   * HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어준다.
   * 예전에는 @RequsetMapping(method=RequestMethod.GET)으로 사용되었다. 이제는 이 프로젝트는 /hello로 요청이 오면 문자열 hello를 반환하는 기능을 가지게 되었다.





#### ControllerTest.java

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilder.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

//1.
@RunWith(SpringRunner.class)
//2.
@WebMvcTest(controllers = Controller.class)
public class ControllerTest {
    //3.
    @AutoWired
    private MockMvc mvc;
    
    //4.
    @Test
    public void hello가_리턴된다() throw Exception {
        String hello = "hello";
        
        //5.
        mvc.perform(get("/hello"))
            //6.
            .andExpect(status().isOk())
            //7.
            .andExpect(content().string(hello));
    }
}
```



1.  @RunWith(SpringRunner.class)
   * 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
   * 여기서는 SpringRunner 라는 스프링 실행자를 사용한다.
   * 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.
2.  @WebMvcTest
   * 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션이다.
   * 선언할 경우 @Controller, @ControllerAdvice 등을 사용할 수 있다.
   * 단, @Service, @Component, @Repository 등은 사용할 수 없다.
   * 여기서는 컨트롤러만 사용하기 때문에 선언한다.
3.  @Autowired
   * 스프링이 관리하는 Bean을 주입 받는다.
4.  private MockMvc mvc
   * 웹 API를 테스트할 때 사용한다.
   * 스프링 MVC 테스트의 시작점이다.
   * 이 클래스를 통해 HTTP GET, POST 등에 대한 API를 테스트 할 수 있다.
5.  mvc.perform(get("/hello"))
   * MockMvc를 통해 /hello 주소로 HTTP GET 요청을 한다.
   * 체이닝이 지원되어 아래와 같이 여러 검증 기능을 이어서 선언할 수 있다.
6. andExpect(status().isOk())
   * mvc.perform의 결과를 검증한다.
   * HTTP Header의 Status를 검증한다.
   * 우리가 흔히 알고 있는 200, 404, 500 등의 상태를 검증한다.
   * 여기선 OK 즉 200 error인지 아닌지를 검증한다.
7.  .andExpect(content().String(hello))
   * mvc.perform의 결과를 검증한다.
   * 응답 본문의 내용을 검증한다.
   * Controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증한다.