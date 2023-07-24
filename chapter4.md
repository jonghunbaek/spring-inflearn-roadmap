# 검증1 - Validation
### 검증
+ Controller의 중요한 역할 중 하나는 HTTP요청이 정상인지 확인하는 검증 역할이다.
+ 클라이언트 검증 : UX를 향상시켜 줌, BUT 보안에 취약
+ 서버 검증 : UX가 좋지 않음, BUT 보안에 강함, 검증 오류를 API응답 결과에 잘 남겨줘야 함
+ 그래서 클라이언트 - 서버 모두 검증을 하는 로직이 필요

### 검증 처리(직접)
+ Map을 활용해 검증 조건에 해당하지 않으면 오류 메시지를 넣어줌
+ 뷰에서도 css를 변경해 일일이 표시를 해줘야 한다. 또 중복처리가 많다.
+ 타입오류에 대한 처리가 불가하다.(Integer타입인데 String타입이 들어온 경우)

### BindingResult
+ 스프링이 제공하는 검증오류를 보관하는 객체
+ ModelAttribute 바로 뒤에 와야 한다.
+ bindingresult는 뷰에 자동으로 같이 넘어감
+ 필드에 대한 오류는 new FieldError, 복합 값에 대한 오류는 new ObjectError로
+ #fiedls로 BindingResult가 제공하는 검증 오류에 접근 가능
+ ObjectError로 생성된 값은 ${#fields.globalErrors()} 와 같이 꺼낼 수 있다.
+ FieldError는  th:errors="*{itemName}(th:if 편의 버전) 로 꺼낼 수 있다. th:errorclass="field-error" 로 에러가 있을 경우 클래스의 값을 설정할 수 있다.
+ 타입 오류 시 BindingResult가 없으면 400오류, 있으면 BindingResult가 해당 에러를 담아서 프로그래머가 처리할 수 있게 도와줌
+ BindingResult는 Errors를 상속받는다(둘다 interface). 실제 넘어오는 구현체는 BeanPropertyBindingResult이다.
```java
public FieldError(String objectName, String field, String defaultMessage) {
		this(objectName, field, null, false, null, null, defaultMessage);
	}

	public FieldError(String objectName, String field, @Nullable Object rejectedValue, boolean bindingFailure,
			@Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage) {

		super(objectName, codes, arguments, defaultMessage);
		Assert.notNull(field, "Field must not be null");
		this.field = field;
		this.rejectedValue = rejectedValue;
		this.bindingFailure = bindingFailure;
	}
```
+ FieldError는 2개의 생성자를 가짐
+ rejectedValue(사용자가 입력한 값), bindingFailure(타입 오류같은 바인딩 실패인지, 검증 실패인지 구분)
+ FieldError는 오류 발생 시 사용자 입력값을 저장하는 기능을 제공한다. - rejectedValue에 저장. 즉, 검증 오류 시 스프링이 FieldError를 새로 생성한다.
+ 타임리프의 th:field는 정상 상황에선 객체의 값을, 오류가 있을 때는 에러의 값을 알아서 출력 해줌

### 오류코드와 메시지 처리
+ FieldError의 codes, arguments 필드 값에 메시지소스를 활용해 값을 넣어 줄 수 있다.
+ rejectValue(필드), reject(글로벌)을 활용해 코드를 간소화 할 수 있다. 오류코드를 입력해주는데 이 오류코드는 errors.properties에 있는 에러코드의 앞 단어만 딴 것이고 이건 messageResolver를 위한 오류 코드이다.
+ MessageCodesResolver의 동작 방식 - 검증 오류코드(여기선 "required)로 메시지 코드들을 찾아줌
+ rejectValue(), reject()는 내부적으로 MessageCodesResolver를 사용, FieldError, ObjectError의 생성자 매개 변수로 오류코드 배열을 받을 수 있다.
+ 기본 메시지 생성 규칙 - 객체오류(코드명.objectName -> 코드명), 필드오류(코드명.objectName.field -> 코드명.field -> 코드명.fieldType -> 코드명)
+ 즉 MessageCodesResolver는 구체적인 것에서 덜 구체적인 것으로 메시지 코드를 만든다. 이를 통해 프로덕션 코드를 건들지 않고 properties에서 메시지만 교체할 수 있다.
+ 위 과정은 ValidationUtils로 간편하게 사용할 수 있다. 단, 공백이나 빈 값 같은 단순한 경우에만 적용 가능
+ Validator 활용 - Validator인터페이스를 구현하고 supports()와 validate()를 오버라이딩하여 사용한다.
+ @InitBinder - Controller가 호출 될 때마다 실행, WebDataBinder로 검증기를 넣어 사용
  
### TIP
+ 부정의 부정으로 코딩하는 것은 지양
+ errors?.containsKey -> errors다음에 '?' errors가 null또는 객체가 없을 때도 널포인트 익셉션을 던지지 않음
