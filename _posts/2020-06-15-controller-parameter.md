---
title: "Controller의 parameter 수집"
layout: single    
read_time: true    
comments: true   
categories: 
 - spring  
toc: true    
toc_sticky: true    
toc_label: contents    
description: controller에서 return 타입에 대한 정리  
last_modified_at: 2020-06-15  
---
<br>

## 파라미터 수집과 변환(@RequestParam)
Controller가 파라미터를 수집하는 방식은 파라미터 타입에 따라 자동으로 변환하는 방식을 사용한다.  
즉, int타입으로 선언된 변수는 자동으로 숫자로 변환하게 된다.  
@RequestParam이 그 중에 하나인데, 변수의 이름과 전달되는 파라미터 이름이 다른 경우 유용하게 사용할 수 있다.  
<br>
<br>
```java
@GetMapping("/requestParam")
public String test(@RequestParam("num") int num) {
  log.info("num: " + num);
  
  return "requestParam";
}
```
<br>
변수명과 파라미터 값이 같으면 굳이 사용할 필요는 없다.  
위와 같이 사용하면 문자열 num을 int타입으로 자동으로 변환을 해주게 된다.   
<br>
<br>

## List, Array 
@RequestParam의 경우 여러가지 값을 받게 되는 경우 코드가 길어지는 문제가 발생하는데, 동일 이름의 파라미터가 여러개 전달되는 경우
List나 Array 타입으로도 받을 수 있다.  
<br>
<br>
```java
@GetMapping("/ListArray")
public String test(@RequestParam("num")ArrayList<String> num) {
  log.info("num: " + num);
  
  return "ListArray";
}
```
<br>
```java
//console 화면
INFO: xxx.xxx.controller.SampleController - num: [1, 2, 3]
```
<br>
배열을 처리한다고 한다면 String[]으로 선언해도 동일하게 처리할 수 있다.   
<br>
<br>
<br>

## Object
전달하는 데이터가 DTO나 VO처럼 여러가지를 처리한다고 하면 객체를 받는 타입으로 정해 한꺼번에 처리가 가능하다.   
```java
@GetMapping("/bean")
public String test(xxxDTO dto) {
  log.info("dto: " + dto);
  
  return "bean";
}
```
<br>
```java
//console 화면
INFO: xxx.xxx.controller.SampleController - dto: xxxDTO(dto=[xxxDTO(num=1, str=a)....객체생성])
```
<br>
브라우저 주소창을 이용해 데이터를 입력하고 엔터를 누르면 위와 같이 콘솔 화면에 []안에 DTO클래스에 정의되어있는 객체들을 처리하는 것을 확인할 수 있다.   
jsp view화면이 정의 안되어있다면 브라우저에서는 에러화면이 뜰 것이다.   
<br>
<br>
<br>

## Model
Model 객체는 JSP 내 컨트롤러에서 생성된 데이터를 담아 전달하는 역할을 한다. jsp view로 전달해야하는 데이터를 담아서 보내줄 수 있다.   
Model 2 방식에서 많이 사용하는 request.setAttribute()와 비슷한 역할을 수행한다.   
<br>
```java
public String test(Model model) {
  model.addAttribute("Time: " + new java.util.Date());
  
  return "test";
}
```
<br>
Model타입은 자동으로 스프링 MVC에 Model 타입의 객체를 만들어주기 때문에 개발자는 필요한 데이터를 담아주기만 하면 모든 작업이 알아서 작동이 된다.   
Model 타입을 사용하는 경우는 controller에서 전달된 데이터 이외의 추가적인 데이터를 가져와야할 때 많이 사용하는데, 예를 들면!
<br>
- 페이지 번호를 파라미터 값으로 받고 실제 데이터를 view로 전달을 하게 될 때
- 파라미터의 처리 후 결과를 전달할 때<br>
<br>
<br>   

이 두가지 경우에 Model 타입을 쓰면 유용하다 할 수 있겠다 :)
<br>
<br>
<br>

### @ModelAttribute?
해당 annotation은 강제로 전달받은 파라미터를 model에 담아서 전달할 때 주로 사용한다. ModelAttribute가 걸린 파라미터는 타입에 관계 없이 무조건 화면으로 전달한다.   
<br>
<br>
```java
@GetMapping("/modelAttrubute")
public String test(xxxDTO dto, @ModelAttribute("page") int page) {
  log.info(dto);
  log.info("page: " + page);
  
  return "/sample/modelAttribute";
}
```
<br>
```java
//브라우저 화면
dto : xxxDTO(num=1, str=a)
page : 1 
```
<br>
어노테이션을 제거하고 기본 자료형으로만 선언하게 된다면 화면까지 전달이 되지 않는다. 그러나 어노테이션이 적용된다면 화면에서 까지
페이지 변수의 값이 전달 되는 것을 확인할 수 있다.  
하지만 해당 타입을 현재는 별로 사용하고 있지 않다고 한다...   
<br>
<br>
<br>
### @RedirectAttribute?
Servlet/JSP 에서 response.sendRedirect()와 같은 기능을 한다고 생각하면 된다.  
이 기능은 일회성으로 데이터를 전달하는 용도로 사용한다.   
<br>
```java
rttr.addFlushAttribute("num", 10);

return "redirect:/xxx";
```
<br>
이렇게 addFlushAttribute(이름, 값)이 화면에 한번만 사용하고 다음에는 사용하지 않는다.   
sendRedirect기능과 동일하기 때문에 return 할 때 "redirect:/" 또는 "forward:/" 등을 할 수 있다.   
<br>
<br>
<br>
<br>

