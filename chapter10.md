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
+ 
