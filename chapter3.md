# 메시지 국제화 
### 개요
+ label, text에 하드코딩되어 있는 단어를 변경할 때 생기는 문제를 해결하기 위한 방법
+ 예를 들어 messages.properties라는 메시지 관리용 파일을 만들고 키=값 형태로 값을 저장하고 html에서 키값을 불러 사용
+ 국제화는 메시지를 국가별 언어로 만든다고 생각하면 된다.

### 스프링 메시지 소스 설정
+ 스프링이 제공하는 MessageSource 인터페이스를 사용 -> 스프링 부트에선 자동으로 빈등록을 해줌(application.properties/spring.messages.basename=messages - 메시지소스 기본값)

### 메시지 소스 활용
```java
    @Override
		public String getMessage(String code, Object[] args, String defaultMessage, Locale locale) {
			// TODO Auto-generated method stub
			return null;
		}
		@Override
		public String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException {
			// TODO Auto-generated method stub
			return null;
		}
		@Override
		public String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException {
			// TODO Auto-generated method stub
			return null;
		}
```
+ MessageSource 객체를 생성해서 사용
+ 메시지 소스를 못찾은 경우엔 NoSuchMessageException이 발생, defaultMessage를 활용해 없을 경우 기본 메시지가 나오도록 설정할 수 있음
+ args는 Object배열을 넘겨준다.

### 타임리프에서 메시지 활용
+ th:text=#{label.item} -> label.item은 messages.properties에서 설정한 값
+ messages.properties에 문자열이아닌 객체가 들어 있을 경우엔 th:text="#{label.item(${item.itemName})}"과 같은 식으로 사용 가능
+ LocaleResolver를 활용해 동적으로 메시지 국제화를 사용할 수 있다.
