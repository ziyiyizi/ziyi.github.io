---
title: swagger
author: ziyi
tags:
  - spring boot
  - swagger
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - swagger
comments: true
---

## 整合swagger
SwaggerConfig.java

```
@Configuration
@EnableSwagger2
@EnableWebMvc
@ConditionalOnExpression("${enable.swagger:false}")
public class SwaggerConfig extends WebMvcConfigurerAdapter {

    @Bean
    public Docket createRestApi() {
        ParameterBuilder header = new ParameterBuilder();
        List<Parameter> pars = new ArrayList<>();
        header.name("X-Token")
                .description("user token")
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false).build();
        pars.add(header.build());
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.javaee.artastic.Artastic.controller"))
                .paths(PathSelectors.any())
                .build()
                .globalOperationParameters(pars);
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("artastic Apis")
                .description("apis in artastic service")
                .contact(new Contact(null, null, null))
                .version("v1")
                .build();
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }

}
```
application.yml需加上如下配置表示启用swagger

```
enable:
  swagger: true
```

pom依赖

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```