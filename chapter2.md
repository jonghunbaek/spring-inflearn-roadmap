# 타임리프 - 스프링 통합과 폼
### 1.스프링 통합
+ 스프링과 통합을 위한 다양한 기능 제공(SpringEL 문법 통합, 편리한 폼관리위한 추가 속성, 폼컴포넌트, 메시지/국제화 기능, 검증/오류 등)

### 2. 입력 폼 처리
+ th:object=""을 통해 form에서 사용할 객체 지정
+ th:field="" - id, name, value 속성을 모두 자동으로 지정, th:field="*{itemName}" 와 같은 식으로 처리 가능
+ 체크 박스의 경우 false전달이 안돼서 hidden 으로 전달 해줘야 하는데 타임리프에선 알아서 해줌
+ **@ModalAttribute을 메서드 위에서 사용하면 코드의 중복을 줄일 수 있다. 단, Controller에 있는 모든 메서드에 일괄 적용되고 컨트롤러 호출 시 마다 생성됨.** 아래의 예시 참고
```java
@ModelAttribute("regions")
    public Map<String, String> regions() {
        Map<String, String> regions = new LinkedHashMap<>();
        regions.put("SEOUL", "서울");
        regions.put("BUSAN", "부산");
        regions.put("JEJU", "제주");
        return regions;
    }
// model.addAttribute("regions", regions); 와 같다.
```
+ 체크 박스가 여러개 일 때 each문을 돌리게 되면 id값에 1,2,3과 같이 필드 변수 뒤에 숫자를 붙여준다.
+ 예를 들어 th:field="${item.regions}" 이때 name값은 같아도 id값은 다르게 생성된다.
+ SpringEL로 ENUM객체에 직접 접근할 수 있다. 즉, 모델에 담아서 보내 줄 필요가 없음 ex) ${T(hello.itemservice.domain.itme.ItemType).values}"
  
