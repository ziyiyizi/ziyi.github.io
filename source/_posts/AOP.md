---
title: aop
author: ziyi
tags:
  - spring boot
  - aop
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - aop
comments: true
---

# AOP

#### 依赖


```
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

#### 切面类
- 类上加入@Aspect 注解
- 使用@Pointcut 定义一个公共的方法，定义切哪个点
- @Before @After @AfterReturning 这三个注解是切的时间点
- 使用@Slf4J 进行日志记录

```
@Aspect
@Slf4J
@Component
public class HttpAspect {
    @Pointcut("execution(com.javaee.artastic.Artastic.controller.TestController.*(..))")
    public void log(){

    }

    @Before("log()")
    public void doBefore(JoinPoint joinPoint){
        ServletRequestAttributes attributes = (ServletRequestAttributes)RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        logg.info("url={}",request.getRequestURL());
        logg.info("method = {}",request.getMethod());
        logg.info("ip = {}",request.getRemoteAddr());
        logg.info("class_method={}",joinPoint.getSignature().getDeclaringTypeName()+"."+ joinPoint.getSignature().getName());
        logg.info("args = {}",joinPoint.getArgs());
    }


    @After("log()")
    public void doAfter(){

    }

    @AfterReturning(pointcut = "log()",returning = "object")
    public void doAfterReturning(Object object){
        logg.info("response = {}",object);
    }
}
```

```
@Aspect
public class GetLoginUserAspect {

    @Autowired
    private TokenManagerService tokenManagerService;
    @Autowired
    private HttpServletRequest request;

    private static Set<String> noTokenMethod;
    private static Set<String> ignoreControllers;

    static {
        noTokenMethod = new HashSet<>();
        noTokenMethod.add("login");
        noTokenMethod.add("health");

        ignoreControllers = new HashSet<>();
        ignoreControllers.add("com.javaee.artastic.Artastic.controller.TestController");
    }

    @Around("execution(* com.javaee.artastic.Artastic.controller.*.*(..)))")
    public Object advice(ProceedingJoinPoint joinPoint) throws Throwable {
        String declaringTypeName = joinPoint.getSignature().getDeclaringTypeName();
        if (ignoreControllers.contains(declaringTypeName)) {
            return joinPoint.proceed();
        }
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        if (noTokenMethod.contains(signature.getMethod().getName())) {
            return joinPoint.proceed();
        }
        String token = StringUtils.getToken(request);
        UserVo userVo = tokenManagerService.getUserVoByToken(token);
        request.setAttribute(ArtasticConst.LOGIN_USER, userVo);
        return joinPoint.proceed();
    }

}
```
- 在启动类注入

```
@SpringBootApplication
public class App extends SpringBootServletInitializer {

    public static void main( String[] args )
    {
    	SpringApplication.run(App.class, args);
    }

    @Bean
    public GetLoginUserAspect setAutoUserAspect() {
        return new GetLoginUserAspect();
    }
}
```

