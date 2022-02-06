# 인프런 - 스프링 MVC2 편 - 백엔드 웹 개발 활용 기술
</br>

:pushpin: Intro

커리큘럼 : 김영한님 스프링 핵심 원리 강의 뿌시기 👊

분량 : 섹션 1개 강의 !





### **Bean Validation**1

```java
@Data
public class Item{
    @NotNull
    private Long id;
 
    @NotBlank
    private String itemName;
 
    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;
 
    // 수정에서는 수량 자유롭게 변경 가능
    private Integer quantity;
 
}
```

### **Controller에 @Validated 추가**

```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, 
BindingResult bindingResult, 
RedirectAttributes redirectAttributes, 
Model model) {
    ...
}
```

### **검증 순서**

① @ModelAttribute 각각의 필드에 타입 변환 시도

1. 성공하면 다음으로

2. 실패하면 typeMismatch 로 FieldError 추가

② Validator 적용

- 바인딩에 성공한 필드만 Bean Validation 적용

- BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.

예)

itemName 에 문자 "A" 입력 타입 변환 성공 itemName 필드에 BeanValidation 적용

price 에 문자 "A" 입력 "A"를 숫자 타입 변환 시도 실패 typeMismatch FieldError 추가

price 필드는 BeanValidation 적용 X

### **에러 코드**

어노테이션 이름으로 에러 코드 생성됨

**① @NotBlank**

NotBlank.item.itemName

NotBlank.itemName

NotBlank.java.lang.String

NotBlank

**② @Range**

Range.item.price

Range.price

Range.java.lang.Integer

Range

### **도메인 객체 vs 별도의 객체**

**① 폼 데이터 전달에 Item 도메인 객체 사용**

HTML Form -> Item -> Controller -> Item -> Repository

- **장점:** Item 도메인 객체를 컨트롤러, 리포지토리 까지 직접 전달해서 중간에 Item을 만드는 과정이

없음

**- 단점:** 간단한 경우에만 적용 가능. 수정시 검증이 중복될 수 있고, groups를 사용해야 함

**② 폼 데이터 전달을 위한 별도의 객체 사용**

HTML Form -> ItemSaveFrom -> Controller -> Item 생성 -> Repository

**- 장점:** 전송하는 폼 데이터가 복잡해도 거기에 맞춘 별도의 폼 객체를 사용해서 데이터를 전달 받을 수

있음. 보통 등록과, 수정용으로 별도의 폼 객체를 만들기 때문에 검증이 중복되지 않음

**- 단점:** 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가됨

**※ Item은 검증 안함. ItemSaveForm과 ItemUpdateForm 검증**

### **API 호출 Validation 실패 시 400 bad request 뜨는 이유**

- > json request를 ItemSaveForm 객체로 변환하지 못함 (컨트롤러 호출도 안됨)

### **API 호출 3가지 경우**

**① 성공 요청:** 성공

**② 실패 요청:** JSON을 객체로 **생성하는 것 자체**가 실패함 (400 bad request)

**③ 검증 오류 요청:** JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함

### **@ModelAttribute vs @RequestBody**

**① @ModelAttribute:** 각각의 필드 단위로 세밀하게 적용. 특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리 가능 (Validator를 사용한 검증도 적용 가능)

**② @RequestBody:** 전체 객체 단위로 적용. 메시지 컨버터의 작동이 성공해서 Item 객체를 만들어야 @Valid , @Validated 가 적용됨. HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후

단계 자체가 진행되지 않고 예외 발생. 컨트롤러도 호출되지 않고, Validator도 적용 불가.
