---
layout:     post
title:      SpringDoc文档工具
subtitle:   SpringDoc
date:       2023-01-05
author:     W
header-img: img/spring-boot.jpg
catalog: true
tags:
    - Spring
    - SpringDoc
    - SpringBoot
    - Swagger
---

## 前言

之前在SpringBoot项目中一直使用的是SpringFox提供的Swagger库，但官网已经有接近两年没出新版本了！SpringDoc是一款可以结合SpringBoot使用的API文档生成工具，基于OpenAPI 3，是一款更好用的Swagger库！值得一提的是SpringDoc不仅支持Spring WebMvc项目，还可以支持Spring WebFlux项目，甚至Spring Rest和Spring Native项目。

## 正文

### 集成

pom文件

```xml
<!--springdoc 官方Starter-->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.14</version>
</dependency>
```

自定义配置文件

```java
/**
 * SpringDoc API文档相关配置
 * @author wwei
 * 2023/1/5 16:37
 */
@Configuration
public class SpringDocConfig {
    @Bean
    public OpenAPI mallTinyOpenAPI() {
        return new OpenAPI()
                .info(new Info().title("标题")
                        .description("简介")
                        .version("版本号")
                        .license(new License().name("许可证").url("地址")))
                .externalDocs(new ExternalDocumentation()
                        .description("关联文档描述")
                        .url("关联文档地址"));
    }
 
    // 文档分组
    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
                .group("brand")
                .pathsToMatch("/brand/**")
                .build();
    }
 
    // 文档分组
    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
                .group("admin")
                .pathsToMatch("/admin/**")
                .build();
    }
}
```

如果需要集成SpringSecurity，需要在SecurityConfig中配置放行白名单

```java
// Swagger的资源路径需要允许访问
.antMatchers(HttpMethod.GET, 
                        "/",   
                        "/swagger-ui.html",
                        "/swagger-ui/",
                        "/*.html",
                        "/favicon.ico",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/swagger-resources/**",
                        "/v3/api-docs/**"
                )
```

然后在OpenAPI对象中通过addSecurityItem方法和SecurityScheme对象，启用基于JWT的认证功能

```java
private static final String SECURITY_SCHEME_NAME = "BearerAuth";

.addSecurityItem(new SecurityRequirement().addList(SECURITY_SCHEME_NAME))
                .components(new Components()
                                .addSecuritySchemes(SECURITY_SCHEME_NAME,
                                        new SecurityScheme()
                                                .name(SECURITY_SCHEME_NAME)
                                                .type(SecurityScheme.Type.HTTP)
                                                .scheme("bearer")
                                                .bearerFormat("JWT")))
```

SpringDoc还有一些常用的配置可以了解下，更多配置可以参考官方文档

```yaml
springdoc:
  swagger-ui:
    # 修改Swagger UI路径
    path: /swagger-ui.html
    # 开启Swagger UI界面
    enabled: true
  api-docs:
    # 修改api-docs路径
    path: /v3/api-docs
    # 开启api-docs
    enabled: true
  # 配置需要生成接口文档的扫描包
  packages-to-scan: com.macro.mall.tiny.controller
  # 配置需要生成接口文档的接口路径
  paths-to-match: /brand/**,/admin/**
```

### 与SpringFox对比

常用Swagger注解对照

| SpringFox                              | SpringDoc                                               |
| -------------------------------------- | ------------------------------------------------------- |
| @Api                                   | @Tag                                                    |
| @ApiIgnore                             | @Parameter(hidden=true)/@Operation(hidden=true)/@Hidden |
| @ApiImplicitParameter                  | @Parameter                                              |
| @ApiImplicitParameters                 | @Parameters                                             |
| @ApiModel                              | @Schema                                                 |
| @ApiModelProperty                      | @Schema                                                 |
| @ApiOperation(value="foo",notes="bar") | @Operation(summary="foo",description="bar")             |
| @ApiParameter                          | @Parameter                                              |
| @ApiResponse(code="404",message="foo") | @ApiResponse(responseCode="404",message="foo")          |

示例


```java
@Tag(name = "TestControllerApi", description = "测试....")
public interface TestControllerApi {

    @Operation(summary = "添加用户",
            description = "根据姓名添加用户并返回",
            parameters = {
                    @Parameter(name = "name", description = "姓名")
            },
            responses = {
                    @ApiResponse(description = "返回添加的用户",
                            content = @Content(mediaType = "application/json",
                                    schema = @Schema(implementation = User.class))),
                    @ApiResponse(responseCode = "400", description = "返回400时候错误的原因")}
    )
    User addUser(String name);

    @Operation(summary = "删除用户",
            description = "根据姓名删除用户",
            parameters = {
                    @Parameter(name = "name", description = "姓名")
            })
    void delUser(String name);
}
```

