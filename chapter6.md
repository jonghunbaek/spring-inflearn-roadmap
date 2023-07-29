# 로그인 처리1 - 쿠키, 세션
### 회원가입, 로그인 기능 구현
+ id, password가 틀리면 글로벌 오류를 구현 -> db까지 접근해야 하니까

### 쿠키를 활용한 로그인 처리
+ 영속쿠키(만료날짜 입력) vs 세션 쿠키(만료날짜 생략 - 브라우저 종료 시)
+ HttpServletResponse를 매개변수로 받아 response에 cookie를 담아준다.
+ 도메인상 어디든 쿠키에 로그인 정보가 담겨있음
+ 쿠키 보안 문제 - 쿠키 값은 임의 변경 가능, 쿠키에 담긴 정보는 외부 유출 가능
+ 대안 - 쿠키에 중요한 값은 담지 말기, 사용자 별 암호화된 임의의 토큰을 노출(서버에서 토큰과 사용자 id를 매핑해서 인식), 토큰의 만료시간을 30분 이하로 유지

### 세션을 활용한 로그인 처리
+ 세션(uuid를 활용해 sessionId 생성)을 사용해 앞서 말한 대안을 처리 - 즉, 서버와 클라이언트는 sessionId라는 랜덤한 토큰 값으로 통신을 하게 된다.
+ 세션 직접 만들기(생성, 조회, 만료)
``` java
@Component
public class SessionManager {

    public static final String SESSION_COOKIE_NAME = "mySessionId";
    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

    /**
     *  세션 생성
     *  sessionId 생성(임의의 추정 불가능한 랜덤 값)
     *  세션 저장소에 sessionId와 보관할 값 저장
     *  sessionId로 응답 쿠키를 생성해 클라이언트에 전달
     */
    public void createSession(Object value, HttpServletResponse response) {
        // id 생성 후 저장
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        // 쿠키 생성
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);
    }

    /**
     * 세션 조회
     */
    public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie == null) {
            return null;
        }
        return  sessionStore.get(sessionCookie.getValue());
    }

    /**
     * 세션 만료
     * @param request
     */
    public void  expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }

    private Cookie findCookie(HttpServletRequest request, String cookieName) {
        Cookie[] cookies = request.getCookies();
        if (cookies == null) {
            return null;
        }
        return Arrays.stream(cookies)
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
}

```


### Servlet 세션을 활용한 로그인 처리
+ HttpSession을 활용해 사
+ request.getSession(true) - 세션이 있으면 반환, 없으면 새로 생성 // request.getSession(false) - 세션이 있으면 반환, 없으면 null
```java
  public String homeLoginV3Spring(
            @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model) {
```
+ 위와 같은 코드를 활용해 세션을 간편하게 관리 할 수 있다.
+ getMaxInactiveInterval(세션 유효시간), creationTime(세션 생성일시), lastAccessedTime(가장 최근 서버에 접근시간, 클라이언트에서 서버로), isNew(새로 생성된 세션인지)
+ 세션의 종료시점 - 사용자가 서버에 가장 최근에 요청한 시간을 기준으로 30분정도 유지하는게 베스트이고 이 방식을 HttpSession에서 사용한다.
```java
#application.properties
server.servlet.session.timeout=1800
```
위 코드를 통해 세션의 타임아웃을 글로벌로 설정할 수 있다.
+ 세션 타임아웃은 LastAccessedTime이후로 timeout시간이 지나면, WAS가 내부에서 해당 세션을 제거한다.

### Tip
+ session은 메모리를 활용한다. -> 꼭 필요한 경우에만 생성
