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
+ url 링크 거는 방법 4가지
```html
<li><a th:href="@{/hello}">basic url</a></li>
<li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
<li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
<li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>
-----------------------------렌더링 후--------------------------------
<li><a href="/hello">basic url</a></li>
<li><a href="/hello?param1=data1&amp;param2=data2">hello query param</a></li>
<li><a href="/hello/data1/data2">path variable</a></li>
<li><a href="/hello/data1?param2=data2">path variable + query parameter</a></li>
```
+ 리터럴 - 타임리프에선 항상 '' 작은 따옴표로 감싸줘야함. 단, 공백없이 쭉 이어진다면 생략가능(몇 가지 조건하에), |문자 + ${변수} | (리터럴 대체)
+ 연산자 - Elvis연산자(th:text="${data}?:'데이터가 없습니다.'" // No-Operation('_'의 경우 타임리프가 실행되지 않는 것 처럼 동작)
+ 속성 값 설정
```html
<h1>속성 설정</h1>
<input type="text" name="mock" th:name="userA" />

<h1>속성 추가</h1>
- th:attrappend = <input type="text" class="text" th:attrappend="class=' large'" /><br/>
- th:attrprepend = <input type="text" class="text" th:attrprepend="class='large '" /><br/>
- th:classappend = <input type="text" class="text" th:classappend="large" /><br/>

<h1>checked 처리</h1>
- checked o <input type="checkbox" name="active" th:checked="true" /><br/>
- checked x <input type="checkbox" name="active" th:checked="false" /><br/>
- checked=false <input type="checkbox" name="active" checked="false" /><br/>

-----------------------------렌더링 후--------------------------------
<h1>속성 설정</h1>
<input type="text" name="userA" />
<h1>속성 추가</h1>
- th:attrappend = <input type="text" class="text large" /><br/>
- th:attrprepend = <input type="text" class="large text" /><br/>
- th:classappend = <input type="text" class="text large" /><br/>

<h1>checked 처리</h1>
- checked o <input type="checkbox" name="active" checked="checked" /><br/>
- checked x <input type="checkbox" name="active" /><br/>
- checked=false <input type="checkbox" name="active" checked="false" /><br/>
```
+ 반복 - th:each="user : ${users}" // th:each="user, userStat : ${users}" 두 번째 파라미터 userStat을 통해 반복의 상태 등 여러 가지 정보를 확인 가능
+ 조건 - th:if // th:unless -> 조건에 맞지 않으면 태그 자체가 날라가는 것 주의
+ 주석 - html주석(<!-- -->) , 타입리프 파서 주석(<!--/* */--!>) - 렌더링 자체가 안됨, 타임리프 프로토타입 주석(<!--/*/ /*/-->) - 타임리프로 렌더링 된 경우에만 주석처리x
