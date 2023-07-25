# Bean Validation
### 개요
+ 매번 검증 로직을 코딩하는 것은 비효율 적이고, 대부분 일반적인 로직이 많다.
+ 이러한 부분을 공통화하여 표준으로 만든 것이 Bean Validation이다.
+ Bean Validation은 구현체가 아니라 JSR-380인 자바 기술 표준으로 인터페이스이다. 주 구현체는 하이버네이트 validation이다.

### Bean Validation 활용
+ @NotBlank(빈값,공백x), @NotNull(null x), @Range, @Max
+ implementation 'org.springframework.boot:spring-boot-starter-validation' 의존성 추가시 SpringBoot가 Bean Validator를 인지하고 사용할 수 있게해준다.
+ LocalValidatorFactoryBean을 글로벌 Validator로 등록, 변수에 @Valid 또는 @Validated만 적용하면 된다.
+ 예시 선언부
```java
public String addItem(@Valid @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model)
```
+ @ModelAttribute로 요청 파라미터 바인딩 -> 성공하면 다음 로직으로, -> 실패하면 FiedlError추가하고 Validator적용
+ 바인딩(타입변환)에 성공한 필드만 BeanValidation 적용
+ BeanValidation 메시지 찾는 순서 - 생성된 메시지 코드 순서(rejectValue처럼)에서 먼저 찾고 없으면 애노테이션의 message속성 출력, 그래도 없으면 라이브러리에서 제공하는 기본값 출력
+ @ScriptAssert() - 복합 검증 시에 사용하나 사용의 불편성 대비 기능 제약이 많아 잘 안씀

### Bean Validation 한계
+ 등록, 수정 등 같은 필드를 가지고 검증을 할 때 조건이 다른 경우
+ 해결 방법
+ 첫째 - groups, 둘째 - dto(Form)를 활용
+ 첫번째 방법(복잡도 + 두번째 방법의 명확함 때문에 잘 사용하지 않음)
``` java
// SaveCheck, UpdateCheck 인터페이스 생성
// 도메인
@NotNull(groups = {SaveCheck.class, UpdateCheck.class})
@Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
private Integer price;

//컨트롤러(@Valid에는 없는 기능)
public String addItem2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
  ...
}
+ 두번째 방법(Dto를 사용 - 도메인 객체와 폼 전송 데이터가 딱 맞지 않기 때문이다.)
