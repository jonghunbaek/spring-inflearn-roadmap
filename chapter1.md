# 타임리프 - 기본기능
### 타임리프 소개
+ SSR용의 뷰 템플릿
+ 순수 HTML을 최대한 유지하는 내추럴 템플릿 - 웹 브라우저에 직접열어도 내용확인 가능. 단, 서버와 연결이 없기 때문에 동적으로 데이터를 얻을 수는 없음
+ 스프링 통합 지원

### 기본기능 소개
+ 텍스트 출력 - 태그안에선 th:text=${data} / 밖에선 [[${data}]]로
+ Escape와 Unescape 사용시 주의 - th:utext , [(${data})]
+ SpringEL 표현식 - user.username(Object), users[0].username(List) == users[0]['username'], userMap['userA'].username(Map)
+ th:with를 통해 지역변수를 사용할 수 있다.
+ 기본 객체들 - request, response, session, servletContext, locale 등
+ 편의 객체들 - param, session, helloBean(컴포넌트에 등록된)
+ 유틸리티 객체들 - https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-utility-objects
```html
<li><a th:href="@{/hello}">basic url</a></li>
<li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
<li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
<li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>
```
