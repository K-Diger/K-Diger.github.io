---

title: Spring Web MVC - Spring MVC
author: 김도현
date: 2023-01-17
categories: [Spring, MVC]
tags: [Spring, MVC]
math: true
mermaid: true

---

# 우리가 자주 사용하던 @Controller의 역할은 어떤 것이 있을까?

1. Component Scan의 대상이 될 수 있다.
2. Spring MVC로부터 애노테이션 기반의 컨트롤러라고 인식하게 할 수 있다.

즉, RequestMappingHandlerMapping은 @RequestMapping 혹은 @Controller가 클래스에 달려있는 경우에 처리하는데 이를 인식할 수 있도록 하는 것이다.

또한 이러한 기능 외의 편리한 기능을 제공하는데 이를 코드로 살펴보면 다음과 같다.

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("/new-form");
    }
}
```

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public String newForm() {
        return "/new-form";
    }
}
```

위와 같이 직접 ModelAndView를 만들고 반환하지 않고 String으로 View 이름만 적어줘도 똑같이 동작하게 된다!

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");

        ModelAndView modelAndView = new ModelAndView("save-result");
        modelAndView.addObject("member", member);

        return modelAndView;
    }
}
```

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/save")
    public String save(@RequestParam("username") String username, Model model) {
        model.addAttribute("member", member);
        return "save-result";
    }
}
```

파라미터를 받는 내용도 애노테이션 기반으로 간단하게 받아올 수 있으며 타입 캐스팅 또한 지원한다.

그리고 ModelAndView를 반환하는 코드 또한 파라미터로 Model을 받아 그 model에 값을 넣고 반환하기만 하면된다.

이 모든 것은 @Controller가 가진 애노테이션 기반으로 MVC를 작성할 수 있게하는 방법의 큰 장점이다.
