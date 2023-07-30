# 예외 처리와 오류 페이지
### 서블릿의 예외 처리
+ 2가지 방식의 예외처리 - Exception(예외), response.sendError(HTTP 상태코드, 오류메시지)
+ 자바 직접 실행 - 예외 발생 시 catch를 못하면 main()의 쓰레드가 종료되고 메시지를 남긴다.
+ 웹애플리케이션 - 사용자 요청별 별도의 쓰레드 생성, catch를 못하면 서블릿 밖으로 예외가 전달될 경우 WAS까지 예외가 전달된다.
+ WAS까지 예외가 전달의 경우 서버 내부에서 처리할 수 없는 오류로 간주하고 500코드를 반환
+ response.sendError를 사용하면 당장 예외가 발생하기 전에 서블릿 컨테이너에게 오류가 발생했다는 것을 알릴 수 있다. 또 상태코드와 함께 오류메시지도 추가할 수 있다.
+ sendError흐름 - WAS(sendError호출기록확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(response.senderror())
+ response내부에 오류가 발생했다는 상태를 저장하고 클라이언트 응답전에 sendError호출여부 확인하고 오류 코드에 따라 페이지를 노출
+ 즉, sendError를 통해 에러코드를 설정할 수 있다.
+ 서블릿 컨테이너 기본 예외처리 화면은 ux가 별로다. 서블릿이 제공하는 오류화면 기능을 사용해서 해결

### 서블릿 예외처리 - 오류페이지 작동원리
+ sendError호출이 있으면 WAS는 오류페이지 정보를 확인해 반환한다. 즉, 위의 sendError의 흐름을 다시 역으로 진행(WAS는 오류페이지 출력을 위해 /error-page/500을 다시 요청)
+ 쉽게 말해 Controller가 2번 요청 되고 이 때문에 클라이언트는 이런 상황을 전혀 알 수 없다.
+ WAS가 오류페이지를 재요청할 때 request에 attribute를 추가해서 넘겨주기 때문에 오류페이지에서 전달된 오류정보를 활용할 수 있다.
+ 오류 발생 시 WAS 내부에서 재호출이 일어나며 필터와 인터셉터를 한번 더 거치게 된다. 이는 비효율적이므로 요청을 구분하기 위해 DispatcherType이라는 추가 정보를 서블릿에서 제공한다.
+ DispatcherType - REQUEST, ERROR, FORWARD, INCLUDE, ASYNC
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
        return filterRegistrationBean;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "*.ico", "/error", "/error-page/**");
    }
}
```

### 스프링 부트 - 오류 페이지
+ 뷰 템플릿 - 정적 리소스(static, pulic) - static/templates/error.html 순으로 에러페이지를 찾는다. 또 구체적인 파일명이 우선순위가 높다.
+ BasicErrorController -> 동적으로 에러페이지를 표현하고 싶을 때 유리, 아래와 같이 정보를 가져올 수 있다.
```html
<ul>
      <li th:text="|timestamp: ${timestamp}|"></li>
      <li th:text="|path: ${path}|"></li>
      <li th:text="|status: ${status}|"></li>
      <li th:text="|message: ${message}|"></li>
      <li th:text="|error: ${error}|"></li>
      <li th:text="|exception: ${exception}|"></li>
      <li th:text="|errors: ${errors}|"></li>
      <li th:text="|trace: ${trace}|"></li>
    </ul>
```
+ 아래와 같이 properties의 설정을 통해 에러메시지 등의 노출여부를 설정할 수 있다.
```
server.error.include-exception=true
server.error.include-message=always
server.error.include-stacktrace=on_param
server.error.include-binding-errors=on_param
```
+ 에러 처리 컨트롤러를 커스텀하고 싶으면 BasicErrorController 또는 ErrorController를 상속받아 해결(ErrorController는 현재 안쓰는 듯)
