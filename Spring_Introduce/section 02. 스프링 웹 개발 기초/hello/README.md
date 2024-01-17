### Section 02. 스프링 웹 개발 기초
자세한 내용은 [[Spring 입문하기] Section02. 스프링 웹 개발 기초 - 안나의 전두엽 어딘가 🧠](https://dksek3050.tistory.com/56)

![spring](https://github.com/An0401na/Spring_Study/assets/99172832/72d12fc8-c3fa-4d54-9001-8aa44c33a5ca)
<br><br>



---

### ➡️ 정적 컨텐츠

-   정적 컨텐츠 : html 을 그대로 화면에 보여주는 것
-   mvc와 템플릿 엔진 : html을 변형하여 보여주는것
-   API : html이 아닌 데이터를 던져 주는 것

### ➡️ MVC와 템플릿 엔진

```
HelloController.java


@Controller
public class HelloController {
 @GetMapping("hello-mvc")
 public String helloMvc(@RequestParam("name") String name, Model model) {
 model.addAttribute("name", name);
 return "hello-template";
 }
}
```

url이 hello-mvc로 요청되면 helloMvc 함수가 실행되게 되고 이때 url에 포함된 name 이라는 파라미터를 넘겨주어 model에 name 이라는 키에 받아온 값을 담아 hello-template.html 로 넘기게 된다.

```
hello-template.html


<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```

hello.html에서는 모델 데이터를 받아 ${name}에 값을 받게 되는데

http://localhost:8080/hello-mvc?name=spring

이러한 .url로 요청을 하게되면 컨트롤러의 name의 값은 spring이 되며 모델에 name이라는 값에 spring을 담고 그걸 html로 넘겨 주게 된다.

### ➡️ API

### **@ResponseBody**

```
HelloController.java

@Controller
public class HelloController {
 @GetMapping("hello-string")
 @ResponseBody
 public String helloString(@RequestParam("name") String name) {
 return "hello " + name;
 }
}
```

@ResponseBody는 

http 통신에 의해 패킷의 body에 값을 직접 담아 보낸다는 의미로 

API는 html이 아닌 데이터를 직접 보내게 되므로 return 값은 데이터가 된다. 위 코드에서는 문자열이다.

```
@Controller
public class HelloController {
     @GetMapping("hello-api")
     @ResponseBody
     public Hello helloApi(@RequestParam("name") String name) {
     	Hello hello = new Hello();
     	hello.setName(name);
     	return hello;
     }
     
     
     static class Hello {
     
     	private String name;
     
   	  	public String getName() {
    	 	return name;
    	 }
     
     	public void setName(String name) {
     		this.name = name;
     	}
     }
}
```

이는 문자열이 아닌 객체를 직접 보내게 되는 것이고

이때 형태는 json 형태로 보내지게 된다.

http://localhost:8080/hello-api?name=spring


-   http 의 body에 문자 내용을 직접 반환하며
-   뷰 리졸버 대신에 httpMessageConverter가 동작하게 되는데
-   문자열 처리는 StringConverter(STringHttpMessageConveter)
-   객체 처리는 JsonConverter(MappingJackson2HttpMessageConverter)가 동작 된다.
