# 로그인 처리 - 필터, 인터셉터
### 서블릿 필터
+ 웹과 관련된 공통관심사는 AOP보단 HTTP의 헤더나 URL의 정보들이 필요하기때문에 서블릿 필터, 스프링 인터셉터를 활용하는 것이 좋다.
+ 필터 흐럼 - HTTP요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
+ 필터는 특정 URL 패턴에 적용할 수 있다.
+ 필터 제한을 통해 로그인/비로그인 사용자를 구분해서 요청을 제한 할 수 있다.
```java
public interface Filter {
  public default void init(FilterConfig filterConfig) throws ServletException {}
  public void doFilter(ServletRequest request, ServletResponse respones, FilterChain chain) throws IOException, ServletException;
  public default void destroy() {}
}
```
+ 필터도 싱글톤이다.

### 서블릿 필터 활용
+ Filter인터페이스 구현해 사용
+ http요처잉 오면 doFilter를 호출
+ chain.doFilter(request, response)를 사용해 다음 필터가 있으면 호출하고 없으면 서블릿을 호출한다. 사용하지 않으면 다음 단계로 진행이 안된다.
+ 스프링 부트에선 FitlerResistrationBean을 사용해 필터를 등록한다.
```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        // 필터는 체인으로 동작하기에 순서가 필요. 숫자가 낮을수록 먼저 동작
        filterRegistrationBean.setOrder(1);
        // 서블릿 url패턴
        filterRegistrationBean.addUrlPatterns("/*");

        return filterRegistrationBean;
    }
}
```
+ http요청시 같은 요청의 로그에 모두 같은 식별자를 자동으로 남기려면 logback mdc를 활용
+ 미인증 사용자인 경우 httpResponse.redirect() 후에 return; 으로 바로 종료
+ 서블릿 필터를 사용해 미인증 사용자는 특정페이지에 접근 못하도록 구현하고 filter에서 관련 로직만 수정하기에 유지보수 간편
+ chain.doFilter()에서 request와 response를 다른 객체로 바꿔서 전달할 수 있다.

### 스프링 인터셉터
+ 인터셉터는 서블릿 필터와 비슷하지만 스프링에서 제공하면 기능이 더 좋다.
+ 스프링 인터셉터 흐름 - HTTP요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤
```java
// Controller 호출
// 더 정확히는 핸들러 어댑터 호출 전
// 반환값이 true면 다음으로 진행시켜!, false면 진행x
default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
}

// Controller 호출 후(Controller에서 예외가 터지면 실행x)
// 정확히는 핸들러 어댑터 이후
default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
    @Nullable ModelAndView modelAndView) throws Exception {
}

// 요청 완료 후(예외와 상관없이 실행)
// 뷰가 렌더링된 이후 호출
default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
    @Nullable Exception ex) throws Exception {
}
```
+ 꼭 필터를 사용해야하는 경우가 아니라면 인터셉터를 사용하는 편이 좋다.

### 스프링 인터셉터 활용ㅇ
