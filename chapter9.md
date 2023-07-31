# API 예외 처리
### 개요
+ BasicErrorController에는 /error 경로를 처리하는 errorHtml(), error() 두 메서드가 있다. 전자는 에러페이지 후자는 json데이터반환
+ BasicErrorController를 확장하면 json오류메시지를 변경할 수 있다. 에러페이지 호출의 경우엔 편리하나 api오류는 처리가 까다로움

### HandlerExceptionResolver
+ 예외가 컨트롤러 밖으로 넘어가면 예외를 해결하고 동작을 새로 정의할 방법을 제공\
+ 사용법은 아래와 같다.
```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

        try {
            if (ex instanceof  IllegalArgumentException) {
                log.info("IllegalArgumentException resolver to 400");
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                return new ModelAndView();
            }
        } catch (IOException e) {
            log.error("resolver ex", e);
        }
        return null;
    }
}

// WebConfig
@Override
public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
    resolvers.add(new MyHandlerExceptionResolver());
}
```
+ 발생한 오류가 형변환이 되는지 확인하고 된다면 예외를 처리해 정상적으로 동작하는 것처럼 만드는 것이 목적이다.
+ 비어있는 ModelAndView를 반환하면 뷰를 렌더링하지 않고 정상흐름으로 서블릿이 반환된다. 지정해서 반환하면 뷰를 렌더링한다. null인 경우엔 다음 ExceptionResolver를 찾아서 반환하고 없음녀 기존에 발생한 예외를 서블릿 밖으로 던지는데 이는 예외를 처리 못하게 되는 것이다.

+ 

### Tip
```java
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
```
+ 클라이언트의 요청헤더에 Accept 값이 application/json일 때 해당 메서드가 호출되게 만든다.
