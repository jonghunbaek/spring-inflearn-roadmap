# API 예외 처리
### 개요
+ api 통신을 위해 json형태로 데이터를 주고 받는데 에러가 발생하면 html페이지 자체로 응답하게 된다.


### Tip
```java
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
```
+ 클라이언트의 요청헤더에 Accept 값이 application/json일 때 해당 메서드가 호출되게 만든다.
