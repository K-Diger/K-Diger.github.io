---

title: Spring 에서 Filter를 커스텀해보자! (QueryDSL, JWT)

date: 2022-08-12
categories: [Spring, Filter, QueryDSL, JWT]
tags: [Spring, Filter, QueryDSL, JWT]
layout: post
toc: true
math: true
mermaid: true

---

# 시나리오

> 유저는 /diger/join 에 회원가입을 할 수 있다.
>
> Param : userName(String), age(int)

<br>

> 유저는 /diger/login 에 로그인을 할 수 있다.
>
> Param : userName(String)
>
> return : AccessToken(String)

<br>

> 유저는 /diger/filtering 에 Get 요청을 할 수 있다.
>
> @RequestHeader : AccessToken
>
> 조건1. 헤당 유저의 토큰에 들어있는 유저의 나이(age)정보가 서버에서 지정한 나이(age)보다 적으면 Get 요청을 할 수 없다.


<br>

> 또한 모든 요청은 필터 단위에서, 데이터베이스 테이블에 카운팅되어, 요청 횟수를 기록할 수 있다.

---

# 필터를 어떻게 둬볼까?

필터를 두 가지를 둘 것이다.

### 첫번째 필터 : Jwt를 검증하고 처리하는 필터 -> CustomJwtFilter

### 두번째 필터 : Api 요청에 대한 카운팅을 하는 필터 -> CustomApiRequestCountFilter

---

## Dto & Controller

### RequestDto.java

    import lombok.AllArgsConstructor;
    import lombok.Builder;
    import lombok.Data;
    import lombok.NoArgsConstructor;

    public class RequestDto {

        @Data
        @Builder
        @AllArgsConstructor
        @NoArgsConstructor
        public static class JoinForm {
            private String userName;
            private int age;
        }

        @Data
        @Builder
        @AllArgsConstructor
        @NoArgsConstructor
        public static class LoginForm {
            private String userName;
        }

    }

<br>

### DigerController.java

    import lombok.RequiredArgsConstructor;
    import org.springframework.web.bind.annotation.*;
    import study.querydsl.entity.Member;
    import study.querydsl.jwt.JwtUtillizer;
    import study.querydsl.repository.member.MemberRepository;

    import java.util.List;

    @RestController
    @RequiredArgsConstructor
    public class DigerController {

        private final MemberRepository memberRepository;
        private final JwtUtillizer jwtUtillizer;

        @PostMapping("/diger/join")
        public void digerJoinMethod(@RequestBody RequestDto.JoinForm joinForm) {

            Member newMember = Member.builder()
                    .age(joinForm.getAge())
                    .username(joinForm.getUserName())
                    .build();

            memberRepository.save(newMember);
        }

        @PostMapping("/diger/login")
        public String digerLoginMethod(@RequestBody RequestDto.LoginForm loginForm) {
            List<Member> checkingMember = memberRepository.findByUsername(loginForm.getUserName());

            if (checkingMember.size() > 0) {
                for (Member requestLoginMember : checkingMember) {
                    return jwtUtillizer.createAccessToken(requestLoginMember);
                }
            }
            return "로그인 실패!";
        }

        @GetMapping("/diger/flitering")
        public String digerFilteredMethod() {
            return "필터링을 거치고 요청이 성공했습니다!";
        }
    }

---

## Repository

### ApiRequestCountRepository

    import org.springframework.data.jpa.repository.JpaRepository;
    import study.querydsl.entity.ApiRequestCount;

    public interface ApiRequestCountRepository extends JpaRepository<ApiRequestCount, Long>, ApiRequestCountRepositoryCustom {}

<br>

### ApiRequestCountRepositoryCustom

    public interface ApiRequestCountRepositoryCustom {

        void increaseApiRequestCount();
    }

<br>

### ApiRequestCountRepositoryCustomImpl

    import com.querydsl.jpa.impl.JPAQueryFactory;
    import lombok.RequiredArgsConstructor;
    import org.springframework.stereotype.Repository;
    import org.springframework.transaction.annotation.Transactional;

    import static study.querydsl.entity.QApiRequestCount.apiRequestCount;

    @RequiredArgsConstructor
    @Repository
    @Transactional
    public class ApiRequestCountRepositoryImpl implements ApiRequestCountRepositoryCustom {

        private final JPAQueryFactory queryFactory;


        public void increaseApiRequestCount() {
            queryFactory
                    .update(apiRequestCount)
                    .set(apiRequestCount.count, apiRequestCount.count.add(1L))
                    .where(apiRequestCount.id.eq(1L))
                    .execute();
        }
    }

---

## Filter

### CustomJwtFilter.java

    import lombok.RequiredArgsConstructor;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Component;
    import study.querydsl.jwt.JwtUtillizer;

    import javax.servlet.*;
    import javax.servlet.http.HttpServletRequest;
    import java.io.IOException;
    import java.util.Enumeration;
    import java.util.List;

    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class CustomJwtFilter implements Filter {

        private List<String> excludedUrls;

        private final JwtUtillizer jwtUtillizer;

        /**
         * init 메서드 : 필터단계에서 초기에 셋팅 해줄 요소
         *
         * @Param : FilterConfig (getFilterName, getServletContext, getInitParameter, getInitParameterNames)
         */

        @Override
        public void init(FilterConfig filterConfig) throws ServletException {

    //        String excludePatternJoin = filterConfig.getInitParameter("/diger/join");
    //        String excludePatternLogin = filterConfig.getInitParameter("/diger/login");
    //
    //        excludedUrls.add(excludePatternJoin);
    //        excludedUrls.add(excludePatternLogin);

            Filter.super.init(filterConfig);
            log.info("CustomFilter Init !!!");

            String filterName = filterConfig.getFilterName();
            ServletContext servletContext = filterConfig.getServletContext();
            String initParameter = filterConfig.getInitParameter("diger");
            Enumeration<String> initParameterNames = filterConfig.getInitParameterNames();

            System.out.println("filterName = " + filterName);
            System.out.println("servletContext = " + servletContext);
            System.out.println("initParameter = " + initParameter);
            System.out.println("initParameterNames = " + initParameterNames);

            log.info("filterName : " + filterName);
            log.info("servletContext : " + servletContext);
            log.info("initParameter : " + initParameter);
            log.info("initParameterName : " + initParameterNames);

            log.info("CustomFilter Init End!!!!!!");
        }

        /**
         *
         * @param request
         * @param response
         * @param chain
         * @throws IOException
         * @throws ServletException
         */

        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

            log.info("CustomJwt 필터 가동!");

            String path = ((HttpServletRequest) request).getServletPath();

            // 필터링을 제외할 URL 이 아닌경우 --> 토큰 검사해야하는 경우
            if (!path.contains("/diger/join") && !path.contains("/diger/login")) {
                log.info("CustomJwt 필터링 중입니다.");

                HttpServletRequest httpServletRequest = (HttpServletRequest) request;

                Integer requestedUserAge = Integer.valueOf(
                        jwtUtillizer.getUserAgeByToken(httpServletRequest.getHeader("AccessToken")));

                log.info("요청한 유저의 나이는 : {} 입니다.", requestedUserAge);

                if (requestedUserAge < 24) {
                    log.info("저보다 나이가 어려서 이용할 수가 없네요");
                } else if (requestedUserAge >= 24) {
                    log.info("저랑 나이가 같거나 크니까 이용할 수 있어요!");
                }
            }

            log.info("CustomJwtFilter 종료");

            chain.doFilter(request, response);
        }

        @Override
        public void destroy() {
            Filter.super.destroy();
        }
    }

<br>

### CustomApiRequestCountFilter

    import lombok.RequiredArgsConstructor;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Component;
    import study.querydsl.repository.studyWithJohn.ApiRequestCountRepository;

    import javax.servlet.*;
    import java.io.IOException;

    @Component
    @RequiredArgsConstructor
    @Slf4j
    public class CustomApiRequestCountFilter implements Filter {

        private final ApiRequestCountRepository apiRequestCountRepository;


        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
            Filter.super.init(filterConfig);
        }

        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

            log.info("Api Count 필터 가동!");

            apiRequestCountRepository.increaseApiRequestCount();

            chain.doFilter(request, response);
        }

        @Override
        public void destroy() {
            Filter.super.destroy();
        }
    }

---

## 뭔가 이상한데...

위와 같은 구현을 마친 후 API 요청을 하면, CustomApiRequestCountFilter가 먼저 동작한다.

나는 우선순위를 준 코드가 없는데 왜 CustomApiRequestCountFilter 보다 먼저 실행이 되는걸까?

