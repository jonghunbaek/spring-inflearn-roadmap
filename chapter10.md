# 스프링 타입 컨버터
### 개요
+ 쿼리스트링으로 들어오는 파라미터는 모두 문자(열)이다. spring을 사용하지 않으면 개별로 타입을 변환해줘야 한다.
+ 스프링에서의 타입 변환 - 확장 가능한 Converters 인터페이스를 제공한다.
```java
public interface Converter<S, T> {

	@Nullable
	T convert(S source);
```
+ 개별로 구현하기에 불편함이 존재. 컨버전 서비스를 통해 해결

### 스프링에서 제공하는 ConversionService
+ 컨버전 서비스는 등록할 때만 명확히 알면 사용할 때는 어떤 메서드를 사용할 지 몰라도 된다.
+ DefaultConversionService는 ConversionService와 ConversionRegistry를 상속 받는데, 전자는 컨버터의 사용에 후자는 컨버터의 등록에 특화된 기능을 가지고 있다.
+ 이렇게 인터페이스 관심사를 분리함으로써 추후 등록 방법 변경에도 컨버터 사용에는 영향이 없고 클라이언트는 꼭 필요한 메서드만 알게 된다. 이는 객체지향 5원칙 중 하나인 ISP(Interface Segregation Principal)이다.
+ 스프링에서 지원하는 컨버전 서비스는 아래와 같이 구현할 수 있다.
```java
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new IntegerToStringConverter());
        registry.addConverter(new StringToIpPortConverter());
        registry.addConverter(new StringToIntegerConverter());
        registry.addConverter(new IpPortToStringConverter());
    }
}
```
+ 컨버터를 추가 등록하면 기본 컨버터보다 우선순위로 실행된다.
+ ${{number}} -> 뷰템플릿은 데이터를 문자로 출력.
+ th:field 컨버전 서비스를 적용한다.

### Formatter
+ Formatter를 상속받아 구현
+ Formatter를 FormatterConversionService에 등록해서 사용하자 - DefaultFormattingConversionService로 구현 가능
+ 하지만 스프링 부트에선 WebConversionService를 내부에서 지원
+ 스프링 기본 지원 포맷터 - 기본 형식 지정으로 필드마다 다른 형식 지정이 어려움
+ 애노테이션으로 원하는 형식을 사용할 수 있도록 지원 - @NumberFormta(pattern = ~~), @DateTimeFormat
