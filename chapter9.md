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
+ ExceptionResolver를 적용하면 예외가 서블릿 컨테이너까지 전달되지 않고 스프링 mvc내에서 다 처리가 된다는 것이 포인트다.

### 스프링이 제공하는 ExceptionResolver
+ HandlerExceptionResolverComposite 등록 순서 - ExceptionHandlerExceptionResolver -> ResponseStatusExceptionResolver -> DefaultHandlerExceptionResolver
+ ExceptionHandlerExceptionResolver(대부분의 api 예외처리), ResponseStatusExceptionResolver(@ResponseStatus(value = HttpStatus.NOT_FOUND), DefaultHandlerExceptionResolver(기본예외 처리)
+ @ExceptionHandler로 예외 처리를 쉽게 만들어줌 -> ExceptionHandlerExceptionResolver
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(IllegalArgumentException.class)
public ErrorResult illegalExHandler(IllegalArgumentException e) {
    log.error("[exceptionHandler] ex", e);
    return new ErrorResult("BAD", e.getMessage());
}

// 매개변수로 받으면 어노테이션엔 생략 가
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandler(UserException e) {
    log.error("[exceptionHandler] ex", e);
    ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
    return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
}

@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
@ExceptionHandler
public ErrorResult exHandler(Exception e) {
    log.error("[exceptionHandler] ex", e);
    return new ErrorResult("EX", "내부 오류");
}
```
+ @ExceptionHandler가 선언돼 있는 컨트롤러에서 예외가 발생하면 @ExceptionHandler가 호출이 되고 이 메서드에 지정된 예외뿐아니라 자식 클래스들의 예외도 잡을 수 있다.
+ @ExceptionHandler에 2가지 이상의 예외를 넣을 수 있고 api예외 처리 뿐 아니라 일반 에러 페이지 호출도 가능하다.

### @ControllerAdvice
+ 대상으로 지정한 컨트롤러에 @ExceptionHandler, @InitBinder 기능을 부여해주는 역할, 지정하지 않으면 글로벌 대상
+ @ControllerAdvice(annotations = RestController.class), @ControllerAdvice("org.example.controllers") 등 전역, 특정 패키지, 특정 클래스를 대상으로 지정할 수 있다.
+ 


### Tip
```java
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
```
+ 클라이언트의 요청헤더에 Accept 값이 application/json일 때 해당 메서드가 호출되게 만든다.
