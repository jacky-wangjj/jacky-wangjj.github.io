---
layout: post
title: SpringBoot开发相关注解
date: 2019-06-09
tags: springboot
---  
### Spring注解
参考链接：           
https://blog.csdn.net/weixin_39805338/article/details/80770472           
https://www.cnblogs.com/xiaoxi/p/5935009.html        



### Jersey注解
#### 声明http请求的请求方式
##### @GET
GET请求（读取、列出、检索单个或资源集合）       

##### @POST
POST请求（新建资源）

##### @PUT
PUT请求（更新现有资源或资源集合）

##### @DELETE
DELETE请求（删除资源或资源集合）

#### @Produces,@Consumes
##### @Consumes
用来指定可以接收client发送过来的数据标识类型（MIME类型），同样可以用于class或者method，也可以指定多个MIME类型，一般用于@PUT,@POST        

接收json格式的数据       
```java
@Consumes(MediaType.APPLICATION_JSON)
```

##### @Produces
用来指定将要返回给client端的数据标识类型（MIME类型），可以可以用于class或者method，一般用于@GET，有数据返回的情况。

返回给前端json格式的数据       
```java
@Produces(MediaType.APPLICATION_JSON)
```

#### 请求参数Annotation
##### @PathParam
用于获取url中path路径中的参数值
```java
@DELETE
@Path("/mqHosts/{mqId}/queues/{queueId}")
@Produces(javax.ws.rs.core.MediaType.APPLICATION_JSON)
@ApiOperation(value = "删除MQ主机下的一个队列")
public Object deleteMqQueue(@Context HttpServletRequest request,
                                @PathParam("mqId") String mqId,
                                @PathParam("queueId") String queueId) {}
// url /mqHosts/1/queues/2    
// mqId=1,queueId=2
```

##### @QueryParam
用于获取url中?后跟的键值对形式的参数
```java
@GET
@Path("/interfaceLimits")
@Produces(javax.ws.rs.core.MediaType.APPLICATION_JSON)
@ApiOperation(value = "查询租户所有计费接口的计费规则")
public Object getAllInterfaceLimitsByTenantId(@Context HttpServletRequest request,
                                              @QueryParam("tenantId") String tenantId) {
// url /interfaceLimits?tenantId=1
// tenantId=1
```

##### @Context
用来解析上下文参数(@Context HttpRequest request)

##### @FormParam
客户端以form(MIME为**application/x-www-form-urlencoded**)的方式提交表单，服务端使用@FormParam解析form表单中的参数

##### @FormDataParam
通常在上传文件的时候，需要@FormDataParam。客户端提交form(MIME为**multipart/form-data**)的方式提交表单，服务端使用@FormDataParam来解析form表单中的参数

文件上传
```java
@POST
@Path("import-excel")
@Consumes(MediaType.MULTIPART_FORM_DATA)
@Produces(MediaType.APPLICATION_JSON)
public ImportResultBean importForExcel(@FormDataParam("file") String fileString,
                                       @FormDataParam("file") InputStream fis,
                                       @FormDataParam("file") FormDataContentDisposition fileDisposition) {
  // TODO
  return ;
}
```

文件下载需要将Response对象的压缩方式
```java
@Produces(MediaType.APPLICATION_OCTET_STREAM)
```

##### @HeaderParam
获取http请求头中的参数值

##### @CookieParam
获取http请求头中cookie中的参数值

##### @DefaultValue
@DefaultValue配合@PathParam、@QueryParam、@FormParam、@FormDataParam、@MatrixParam、@HeaderParam、@CookieParam等使用，如果请求指定的参数中没有值，就使用@DefaultValue中的值为默认值。注意：@DefaultValue指定的值在解析过程中出错（比如@DefaultValue("test") @QueryParam("age") int age），将返回404错误。

##### @BeanParam
如果传递的参数较多，可以自己写个bean，bean中的字段使用@PathParam、@QueryParam、@FormParam、@FormDataParam、@MatrixParam、@HeaderParam、@CookieParam来注解。而在resouces中具体方法参数中就可以使用@BeanParam来注解这个自定义的bean

##### @Encoded
禁止解码，客户端发送的参数是什么样的，服务器接收到的参数就是什么样。

#### @Path
##### 标注class
表明该类是个资源类。凡是资源类，必须使用@Path注解，不然jersey无法扫描到该资源类。

##### 标注method
表示具体的请求资源的路径。

### lombok注解

```xml
    <!-- Tool for simplifying code -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
```

##### @Data
setting和getting方法

##### @AllArgsConstructor
全参构造器

##### @NoArgsConstructor
无参构造器
