---
title: "Spring mvc"    
layout: single    
read_time: true    
comments: true   
categories: 
 - spring  
toc: true    
toc_sticky: true    
toc_label: contents    
description: Spring mvc의 java설정과 annotation에 대한 정리  
last_modified_at: 2020-06-14   
---
스프링에서 xml 설정보다는 가독성이 높고 관리하기가 편한 java configuration으로 설정하는 것이 대세라고 한다.  
그래서 나도 한번 해봤다!! 
<br>
<br>
<br>
## ServletConfig 설정하기
webconfig와 rootconfig는 설정을 했으니, servletconfig를 설정해보았다.
<br>
<br>
```java
//ServletConfig.java

@EnableWebMvc //mvc 구성에 필요한 빈(객체)를 자동으로 생성해주는 annotation
@ComponentScan(basePackages = { "org.zerock.controller", "org.zerock.exception"})
public class ServletConfig implements WebMvcConfigurer{ //webMvcConfigurer: 생성된 빈을 커스터마이징 가능하게 해줌
	
	//기존 servlet-context.xml 내용 작성하기
	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		InternalResourceViewResolver bean = new InternalResourceViewResolver();
			bean.setViewClass(JstlView.class);
			bean.setPrefix("/WEB-INF/views/");
			bean.setSuffix(".jsp");
			registry.viewResolver(bean);
	}
	
	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
	}

```
<br>
@ComponentScan으로 controller 패키지와 exception예외 처리 패키지를 스캔하도록 설정해두었다. 
<br>
<br>
**configureViewResolvers method**는 prefix로 view의 경로를 잡아주고 하위에 포함된 jsp파일을 동작할 수 있도록 suffix가 지정되어있다. 
controller가 반환한 결과를 어떤 View를 통해 처리하는 것이 좋을지 해석하는 역할을 담당한다.  
<br>
<br>
**두번째 addResourcesHandlers**는 정적 리소스(static resources-html, css등)로, /resources로 경로를 잡아주고 user가 server에 요청을 할 때 맞는 url을 맵핑해 html화면을 보여주거나 http cache를 조절하는 기능까지 담당한다.   
여기서는 캐시 조절까진 아니고 /resources로 경로만 잡아주는 용도로 사용하고있다.  
<br>
<br>
## Spring의 로딩 구조
프로젝트의 로딩은 WebConfig에서 시작을 하게 된다.
<br>
<br>
![webconfig 이미지]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/webconfigmvc.PNG){: .align-center}  
<br>
- webconfig에서 rootconfig와 servletconfig의 경로를 설정해두었고, rootconfig의 Bean설정들이 먼저 동작하게 된다.<br><br>
- 스프링 영역(context)에 Bean(객체)들이 생성되어 서로의 의존성이 처리가 되면 스프링에서 사용하는 DispatcherServlet이 동작한다.<br><br>
- DispatcherServlet이 servletconfig를 로딩하여 해당 파일에 등록된 Bean을 기존의 생성된 객체들과 연동을 시켜 최종적으로 로딩이 완료된다.<br><br>  
<br>
<br>
### DispatcherServlet?
위의 코드로 보면 모든 Request는 DispatcherServlet이 받도록 처리되고 있으며 HandlerMapping에게 요청받은 Request의 맞는 컨트롤러를 
찾게 한다. 요청을 모두 처리한 후에 View로 응답을 보낼 때도 DispatcherServlet이 담당을 하게 된다.  
<br>
<br>

이렇게 모든 Request를 DispatcherServlet이 받게 하는 패턴을 Front-Controller 라고 부른다는데, Request의 처리에 대한 분배가
정해진 방식으로만 동작하도록 구성하기 때문에 구조를 엄격하게 만들 수 있다고 한다.  
<br>
<br>
<br>
이처럼 스프링MVC를 이용하게 되면 Servlet/JSP에서 많이 사용하는 HttpServletRequest/HttpServletResponse를 사용할 필요성이 현저히 줄어들고 
Annotation과 xml(또는 java configuration)으로 개발이 가능하다! :) 
<br>
<br>
<br>
<br>








