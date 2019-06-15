---
layout: post
title: Swagger使用指南
date: 2019-06-09
tags: springboot
---  
### swagger简介
[swagger官网](https://swagger.io/): https://swagger.io/       
Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。       


https://www.imooc.com/article/71842


- maven依赖      
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
```

### swagger配置
```java
package com.swaggerTest;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * Swagger2配置类
 * 在与spring boot集成时，放在与Application.java同级的目录下。
 * 通过@Configuration注解，让Spring来加载该类配置。
 * 再通过@EnableSwagger2注解来启用Swagger2。
 */
@Configuration
@EnableSwagger2
public class Swagger2 {

    /**
     * 创建API应用
     * apiInfo() 增加API相关信息
     * 通过select()函数返回一个ApiSelectorBuilder实例,用来控制哪些接口暴露给Swagger来展现，
     * 本例采用指定扫描的包路径来定义指定要建立API的目录。
     *
     * @return
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.swaggerTest.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    /**
     * 创建该API的基本信息（这些基本信息会展现在文档页面中）
     * 访问地址：http://项目实际地址/swagger-ui.html
     * @return
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("更多请关注https://jacky-wangjj.github.io/")
                .termsOfServiceUrl("https://jacky-wangjj.github.io/")
                .contact("sunf")
                .version("1.0")
                .build();
    }
}
```

### swagger注解
@Api：用在类上，说明该类的作用。

@ApiOperation：注解来给API增加方法说明。

@ApiImplicitParams : 用在方法上包含一组参数说明。

@ApiImplicitParam：用来注解来给方法入参增加说明。

@ApiResponses：用于表示一组响应

@ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息       
|----- code：数字，例如400
|----- message：信息，例如"请求参数没填好"
|----- response：抛出异常的类   

@ApiModel：描述一个Model的信息（一般用在请求参数无法使用@ApiImplicitParam注解进行描述的时候）
|----- @ApiModelProperty：描述一个model的属性       

@ApiImplicitParam的参数说明：

| 参数                         | 说明                |
| :--------------------------: | --------------------|
|paramType：指定参数放在哪个地方 | header：请求参数放置于Request Header，使用@RequestHeader获取|
|                  |query：请求参数放置于请求地址，使用@RequestParam获取|
|                  |path：（用于restful接口）-->请求参数的获取：@PathVariable|
|                  |body：（不常用）|
|                  |form（不常用）|
|name：参数名 |  |
|dataType：参数类型 | |
|required：参数是否必须传  |  true / false
|value：说明参数的意思 | |
|defaultValue：参数的默认值 | |

- paramType会直接影响程序的运行期，如果paramType与方法参数获取使用的注解不一致，会直接影响参数的接收。      

#### API接口实例
```java
package com.swaggerTest.controller;

import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;

/**
 * 一个用来测试swagger注解的控制器
 * 注意@ApiImplicitParam的使用会影响程序运行，如果使用不当可能造成控制器收不到消息
 *
 * @author SUNF
 */
@Controller
@RequestMapping("/say")
@Api(value = "SayController|一个用来测试swagger注解的控制器")
public class SayController {

    @ResponseBody
    @RequestMapping(value ="/getUserName", method= RequestMethod.GET)
    @ApiOperation(value="根据用户编号获取用户姓名", notes="test: 仅1和2有正确返回")
    @ApiImplicitParam(paramType="query", name = "userNumber", value = "用户编号", required = true, dataType = "Integer")
    public String getUserName(@RequestParam Integer userNumber){
        if(userNumber == 1){
            return "张三丰";
        }
        else if(userNumber == 2){
            return "慕容复";
        }
        else{
            return "未知";
        }
    }

    @ResponseBody
    @RequestMapping("/updatePassword/{userId}")
    @ApiOperation(value="修改用户密码", notes="根据用户id修改密码")
    @ApiImplicitParams({
        @ApiImplicitParam(paramType="query", name = "password", value = "旧密码", required = true, dataType = "String"),
        @ApiImplicitParam(paramType="query", name = "newPassword", value = "新密码", required = true, dataType = "String")
    })
    public String updatePassword(@ApiParam(value = "user's id", required = true, dataType = "Integer")
            @PathParam(value = "userId") Integer userId,
            @RequestParam(value="password") String password,
            @RequestParam(value="newPassword") String newPassword){
      if(userId <= 0 || userId > 2){
          return "未知的用户";
      }
      if(StringUtils.isEmpty(password) || StringUtils.isEmpty(newPassword)){
          return "密码不能为空";
      }
      if(password.equals(newPassword)){
          return "新旧密码不能相同";
      }
      return "密码修改成功!";
    }
}
```

#### Model实例
```java
import com.lenovo.medicinealert.common.configuration.Constants;
import com.lenovo.medicinealert.common.model.BaseEntity;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.validation.constraints.NotNull;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.Size;

/**
* @author wangjj17@lenovo.com
* @date 2019/6/6
*/
@Data
@NoArgsConstructor
@AllArgsConstructor
@ApiModel(value = "MenuUpdate", description = "菜单")
public class MenuUpdate implements BaseEntity {
private static final long serialVersionUID = 1524191037779868928L;

@ApiModelProperty(name = "name", value = "菜单名", required = true)
@NotNull
@Size(min = 1, max = 64, message = "Field name cannot be empty. The length of name should not be more than 64 symbols.")
@Pattern(regexp = Constants.NAME_PATTERN_REGEX, message = "Name's format isn't correct.")
private String name;

@ApiModelProperty(name = "description", value = "描述")
@Size(max = 100, message = "The length of description should not be more than 100 symbols.")
private String description;

@ApiModelProperty(name = "url", value = "菜单url", required = true)
@Pattern(regexp = Constants.URL_PATTERN_REGEX, message = "Invalid resource url.")
private String url;

@ApiModelProperty(name = "parentMenuId", value = "父菜单id")
private Long parentMenuId;

@ApiModelProperty(name = "level", value = "优先级", required = true)
@NotNull
@Min(0)
@Max(10)
private Integer level;
}
```

### 通过SwaggerHub编写接口文档

https://blog.csdn.net/qq_32589355/article/details/81089954
