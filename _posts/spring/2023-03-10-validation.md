---
published: true
title : "SpringBoot - Validation"
categories:
  - Spring

tags:
  - Spring

toc: true
toc_sticky: true
sidebar:
  nav: "sidebar-category"
---

# 들어가며
&nbsp; 우리는 다양한 환경에서 Form을 등록한다. 회원가입이든 포스팅이든, 항목의 등록이든, 많은 환경에서 폼을 전송하고, 그 과정에서 우리는 안에 들어있는 값들이 적절한지에 대해서 확인해야 할 필요가 있다.<br>
<br>


물론 **Validation** 이란 프론트단과 서버단 모두에서 적용을 해야 할것이다. 프론트 단에서 단순히 input 태그에 값을
입력하지 않으면, action이 일어 나지 않도록 하는 간단한 방법이 있을것이다. <br><br>
하지만 프론트단의 Valid의 경우 사용자의 직접 조작에 의해 강제로 request시킬수 있는 방법들이 존재하기 때문에, 취약점이 분명 존재한다. 
<br>
따라서 사용자의 편의성을 고려하되, 적절한 값을 Valid 하기 위해서는 Front단과 Server 단에서 각각 적절한 Validation 과정을 진행해야만 한다.<br><br>

아래부터는 Validation의 레거시 코드에서부터 스프링부트에서 진행하는 간단한 방법까지의 발달 과정을 정리 해 볼것이다. 해당 내용은 **인프런 이영한님의 스프링 MVC 2편** 내용을 바탕으로 하고 있다.

<br>
<br>

## Validation 구현 - 1단계(Error Map)

&nbsp; 서버단에서 Validation을 담당해야 하는 곳은 Controller이다.
간단한 Post 요청맵핑을 기준으로 이야기해보자.<br><br>

&nbsp; 1단계는 errors 라는 맵을 만들어 검증로직 실행후 오류가 있다면, 값을 담아 리턴한다.<br><br>

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model){

    //검증 오류 결과를 보관
    Map<String, String> errors = new HashMap<>();

    //검증 로직
    if(!StringUtils.hasText(item.getItemName())){
        errors.put("itemName", "아이템 이름은 필수 입니다.");
    } // 이름이 없다면, errors에 put 하기.
    if(item.getPrice() == null || item.getPrice()<1000 || item.getPrice()>1000000){
         errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
    }
    
    //검증에 실패하면 다시 입력 폼으로 return
    if(!errors.isEmpty()){
        model.addAttribute("errors", errors);
        return "validation/v1/addForm";
    }

    //성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v1/items/{itemId}";


}
```

<br>
<br>

## Validation 구현 - 2단계(BindingResult)
&nbsp; 위와 동일한 환경에서, errors라는 직접 생성한 Map이 아닌 스프링 프레임워크에서 제공하는 BindingResult 객체에 담아 리턴한다.

* BindingResult는 검증 오류가 발생할 경우 오류 내용을 보관하는 스프링 프레임워크에서 제공하는 객체이다.
* 파라미터 값 중 BindingResult는 Validation할 객체 바로 뒤에 위치해야 한다.(위치에 주의 할것!)

```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    //검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수 입니다."));
    }
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999개 까지 허용합니다"));
    }


    //검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) { 
        //!errors.isEmpty()이 bindingResult.hasErrors()로 바뀐것에 주의하자
            log.info("errors = {} ", bindingResult);
            return "validation/v2/addForm";
    }

    //성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

<br>
이렇게 사용할 시의 문제점은, 사용자의 잘못된 입력값이 저장되지 않고, form 데이터 자체가 날라가 버린다.
<br>
<br>


## Validation 구현 - 3단계(BindingResult + 사용자 입력값 저장)

&nbsp; 사용자 입력값 저장을 알기 위해서는 FieldError의 두가지 생성자를 알아야 한다.


* public FieldError(String objectName, String field, String defaultMessage);
* public FieldError(String objectName, String field, @Nullable Object 
rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
Object[] arguments, @Nullable String defaultMessage)

첫번째 생성자의 경우 2단계에서 우리가 사용했던 생성자이며, 두번쨰 생성자를 살펴보면 Object 형식으로 rejectedValue를 저장할수 있다.<br>


간단하게 비교해보기 위해 검증로직 부분만 가져왔다.


``` java
@PostMapping("/add")
public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    //검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null,null, "상품 이름은 필수 입니다."));
    }
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item", "price" ,item.getPrice(), false,null,null, "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(),false,null,null,"수량은 최대 9,999개 까지 허용합니다"));
    }

```

위의 new FieldError 에서 item.getItemName() 파라미터 부분이 거절된 사용자의 입력값을 담는 부분이다.
<br>
<br>


## Validation 구현 - 4단계(오류 메시지 처리 1단계)
&nbsp; 위의 Validation에서는 오류가 났을시, FieldError 객체에 defalutMessage를 직접적으로 하드코딩해주고 있다. 하지만 비슷한 내용의 오류 검증 코드가 무진장 많다면??? 일일이 다 적기에는 너무 피곤한 작업이다.<br>
<br>
아직 다루지는 않았지만, 스프링의 메시지 기능을 이용해보자.
application.properties 에 메시지 기능을 추가한다.
<br>

```java:application.properties
spring.messages.basename=messages,errors
```

<br>
errors.properties 파일을 생성해 내용을 추가한다.
<br>


```java:errors.properties
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}

```

<br>


이번에도 검증로직 까지만 가져왔다, 뒷 부분은 위와 동일하니, 동작 안한다고 내탓하기 없기.

```java
@PostMapping("/add")
public String addItemV3(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    log.info("objectName={}", bindingResult.getObjectName());
    log.info("target={}", bindingResult.getTarget());

    //검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, new String[]{"required.item.itemName"},null,null));
    }
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item", "price" ,item.getPrice(), false,new String[]{"range.item.price"},new Object[]{1000, 1000000}, null));
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(),false,new String[]{"max.item.quantity"},new Object[]{"9999"},null));
    }
}
```

상단의 FieldError 생성자에서 봤듯, String[] 배열로 코드를 넣는 파라미터에 우리가 메시지 기능으로 만들어 뒀던 코드를 넘기고. 코드 속 arguments가 있다면, 치환할 값을 그다음 오브젝트로 넘기게 되어있다.

<br>

## Validation 구현 - 5단계(오류 메시지 처리 2단계)
&nbsp; 여기까지 잘 따라 왔다면, Validation에 대한 감이 좀 잡혔을 것 같다. 그치만 코드가 너무 복잡해.. 앞으로는 저 코드를 간단하게 줄이는 과정에 집중해보자.

* BindingResult는 이미 검증해야 할 객체인 target 바로 다음에 위치하기 때문에, 본인이 검증해야 할 객체인 target을 알고 있다.
* BindingResult가 제공하는 rejectValue(). reject() 메소드를 사용해보자


다시한번 말하지만, 고대로 복붙하면 오류난다. 검증로직 외 부분은 다 생략해버린 상태이다.

```java
@PostMapping("/add")
public String addItemV4(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    //검증로직
    if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.rejectValue("itemName", "required");
    }
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
    }
}
```

이렇게 변경해도 정상적으로 동작한다. 우리는 errors.properties에 있는 값을 제대로 넘겨주지 않았는데 어떻게 된 것일까?

rejectValue() 메소드를 확인해보자

    void rejectValue(@Nullable String field, String errorCode,
@Nullable Object[] errorArgs, @Nullable String defaultMessage);

    bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)
    
    range.item.price=가격은 {0} ~ {1} 까지 허용합니다.

    target인 item을 알고 있으니, ragne.item.price에 접근이 가능하다.

예를 들어
```java
//Level1
required.item.itemName: 상품 이름은 필수 입니다.
//Level2
required: 필수 값 입니다.
```

메시지에 이렇게 두가지를 설정해 두면, 세부적인걸 우선으로 검색하고 그것이 없다면, 좀더 공용적인 requried에 접근해 해당하는 메시지를 가져온다.

    FieldError rejectValue("itemName", "required")
    다음 4가지 오류 코드를 자동으로 생성
    required.item.itemName
    required.itemName
    required.java.lang.String
    required

<br>
<br>

## Validation 구현 - 5단계(Validator 분리)
&nbsp; 컨트롤러가 너무 길고 복잡한 감이 있따. 복잡한 검증 로직을 별도로 분리하여, 컨트롤러 단을 깔끔하게 해보자.

검증로직 부분을 ItemValidator.java의 별도의 클래스로 생성한다.

```java : ItemValidator.java

@Component
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
        //item == clazz
        //item == subItem
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;

        if (!StringUtils.hasText(item.getItemName())) {
                errors.rejectValue("itemName", "required");
        }
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
                errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
                errors.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        // 특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
                int resultPrice = item.getPrice() * item.getQuantity();
                if (resultPrice < 10000) {
                        errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
                }
        }



    }
}

```

<br>
<br>

컨트롤러에서 ItemValidator를 호출해보자.

```java
@PostMapping("/add")
public String addItemV5(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        itemValidator.validate(item, bindingResult);

        //검증에 실패하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
                log.info("errors = {} ", bindingResult);
                return "validation/v2/addForm";
        }

        //성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
}
```

정상적으로 잘 작동된다.

## Validation 구현 - 6단계(Validator 분리/간소화)
&nbsp; Validator를 직접 호출하는것이 아닌, 스프링의 Validator 인터페이스를 사용해서 호출해서 간단하게 만들어보자.

컨트롤러 상단에 다음을 추가해서 밸리데이터를 호출하자.

```java
@InitBinder
public void init(WebDataBinder dataBinder) {
 log.info("init binder {}", dataBinder);
 dataBinder.addValidators(itemValidator);
}
```

이런식으로 검증기를 추가하면, 해당 컨트롤러에서 검증기를 자동 적용할수 있다.
@ModelAttribute 앞에 @Validated 어노테이션만 사용하면, 해당 검증기를 바로 사용할수 있음.

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //검증에 실패하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
                log.info("errors = {} ", bindingResult);
                return "validation/v2/addForm";
        }

        //성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
}

```

@Validated 어노테이션이 붙으면 WebDataBinder에 등록한 검증기를 찾아서 실행한다. 여러 검증기를 등록한다면 그 중 어떤 검증기가 실행되어야 할지 구분이 필요하다. 이때 ItemValidater가 상속한 Validator 인터페이스의 supports() 가 작동하여, 해당 도메인이 맞는지 검증하게 된다.

<br>
<br>

글로벌 설정의 경우 main메서드가 있는 class에 WebMvcConfigurer를 상속받게 한후, getValidator() 메소드를 오버라이드 시키고, 해당 검증기를 등록하면 된다.

```java
@SpringBootApplication
public class ItemServiceApplication implements WebMvcConfigurer {
    
    public static void main(String[] args) {
        SpringApplication.run(ItemServiceApplication.class, args);
    }
 
    @Override
    public Validator getValidator() {
        return new ItemValidator();
    }
}

```

<br>
<br>

## 끝내며

&nbsp; Error Map부터시작해서 BindingResult 그리고 메시지, 축약 버전 까지 업그레이드 과정을 하나하나 공부하며 정리해보았습니다.
머릿속에 멤돌던 막연한 지식들을 정리할수 있는 기회가 되었으면 좋겠습니다.
오타나 틀린부분이 있을수도 있으니, 양해 바랍니다.





























