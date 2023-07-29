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

### 스프링 인터셉터 활용
+ preHandle에서 생성한 uuid 값을 다른 메서드에서 활용하기 위해선 request.setAttribute로 담아 사용해야한다. LogInterceptor도 싱글톤으로 사용되기 때문에 멤버 변수로 활용하면 문제가 생긴다.
+ 서블릿 필터와 마찬가지로 WebConfig에 빈으로 등록해 사용한다. 차이점은 WebMvcConfigurer를 구현해야 하는 것이다.
```java
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");
    }
}
```
+ 핸들러 정보는 @Controller와 같은 HandlerMethod나 정적리소스와 같은 ResourceHttpRequestHandler로 다르게 넘어오기 때문에 분기 처리가 필요
```java
// @RequestMapping : HandlerMethod
// 정적 리소스 : ResourceHttpRequestHandelr
if (handler instanceof HandlerMethod) {
    HandlerMethod hm = (HandlerMethod) handler; // 호출할 컨트롤러 메서드의 모든 정보가 포함
}
```
+ addPathPatterns , excludePathPatterns로매우정밀하게 URL패턴을지정할 수 있다.
+ 인증 체크 예외 url또한 webconfig에서 설정가능 하다.
```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
registry.addInterceptor(new LogInterceptor())
	.order(1)
	.addPathPatterns("/**")
	.excludePathPatterns("/css/**", "/*.ico", "/error");

registry.addInterceptor(new LoginCheckInterceptor())
	.order(2)
	.addPathPatterns("/**")
	.excludePathPatterns("/", "/members/add", "/login", "/logout", "/css/**", "/*.ico", "/error");
}
```
+ 확실히 서블릿 필터에 비해 간결해진 코드와 webconfig에서 공통적으로 세밀한 설정이 가능능

### ArgumentResolver 활용한 로그인 처리
+ @Login 이라는 사용자 설정 애노테이션을 만들어주고 해당 값을 담아주고 싶은 변수 앞에 붙여준다.
```java
// 파라미터에만 적용
@Target(ElementType.PARAMETER)
//  리플렉션등을 활용할 수 있도록 runtime까지 애너테이션이 남아있어야 하기 때문
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {}

// HomeController
public String homeLoginV3ArgumentResolver(@Login Member loginMember, Model model)
```
+ 현재 상태에선 @Login을 @ModelAttribute와 같이 인식하기 때문에 HandlerMethodArgumentResolver을 구현하는 구현체를 만들어준다.
```java
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {

    // @Login 애노테이션이 있으면서 Member 타입이면 해당 ArgumentResolver가 사용된다.
    //supports는 한번만 호출 - 캐시에 값이 저장돼 있기 때문
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        log.info("supportsParameter 실행");

        // 로그인 애너테이션이 이 파라미터에 붙어 있는지
        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
        // 여기선 타입이 Member가 된다.
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());

        return hasLoginAnnotation && hasMemberType;
    }

    // 컨트롤러 호출 직전에 호출되어 필요한 파라미터 정보를 생성. 여기선 세션에 있는 로그인 회원 정보인 member객체를 찾아서 반환. 이후 스프링이 컨트롤러의
    // 메서드를 호출하면서 여기에서 반환된 member객체를 파라미터에 전달
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        log.info("resolvereArgument 실행");

        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        HttpSession session = request.getSession(false);
        if (session == null) {
            return null;
        }

        return session.getAttribute(SessionConst.LOGIN_MEMBER);
    }
}
```
+ 마지막으로 WebConfig에 해당 ArgumentResolver를 등록해준다.
```java
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new LoginMemberArgumentResolver());
    }
```
+ ArgumentResolver를 활용하면 공통작업이 필요할 때 Controller를 더욱 편리하게 사용할 수 있다.

### 서블릿 필터 & 스프링 인터셉터
+ 필터와 인터셉터는 웹과 관련된 공통 관심사를 해결하기 위한 기술이다.
+ 필터보단 인터셉터가 확실히 편리. 무조건 필터를 사용해야되는 상황이 아니라면 인터셉터를 사용하자.
  

### Tip
? 한 문자 일치
* 경로(/) 안에서 0개 이상의 문자 일치 
** 경로 끝까지 0개 이상의 경로(/) 일치
{spring} 경로(/)와 일치하고 spring이라는 변수로 캡처
{spring:[a-z]+} matches the regexp [a-z]+ as a path variable named "spring" 
{spring:[a-z]+} regexp [a-z]+ 와 일치하고, "spring" 경로 변수로 캡처
{*spring} 경로가 끝날 때 까지 0개 이상의 경로(/)와 일치하고 spring이라는 변수로 캡처
/pages/t?st.html — matches /pages/test.html, /pages/tXst.html but not /pages/ 
toast.html
/resources/*.png — matches all .png files in the resources directory
/resources/** — matches all files underneath the /resources/ path, including / 
resources/image.png and /resources/css/spring.css
/resources/{*path} — matches all files underneath the /resources/ path and 
captures their relative path in a variable named "path"; /resources/image.png 
will match with "path" → "/image.png", and /resources/css/spring.css will match 
with "path" → "/css/spring.css"
/resources/{filename:\\w+}.dat will match /resources/spring.dat and assign the 
value "spring" to the filename variable

+ https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html
