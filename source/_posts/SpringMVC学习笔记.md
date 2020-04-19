---
title: SpringMVC学习笔记
date: 2020-04-19 12:14:59
tags:
	- SpringMVC
categories: 学习笔记
---

## 二、SpringMVC

### （一）概述

SpringMVC 是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级 Web 框架，属于SpringFrameWork 的后续产品，已经融合在 Spring Web Flow 中。 

SpringMVC 已经成为目前最主流的MVC框架之一，并且随着Spring3.0 的发布，全面超越 Struts2，成为最优秀的 MVC 框架。它通过一套注解，让一个简单的 Java 类成为处理请求的控制器，而无须实现任何接口。同时它还支持 RESTful 编程风格的请求 。

三大组件：处理器映射器  处理器适配器  视图解析器 

<!--more-->

### （二）流程图示

#### 1.web请求概览

![](http://image.jiruby.cn/2020-04-19%20123500.jpg)

#### 2.mvc内部流程

##### 2.1 图示版

![](http://image.jiruby.cn/2020-04-19%20122100.jpg)

##### 2.2 文字版

①用户发送请求至前端控制器DispatcherServlet。

②DispatcherServlet收到请求调用HandlerMapping处理器映射器。

③处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

④DispatcherServlet调用HandlerAdapter处理器适配器。

⑤HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。

⑥Controller执行完成返回ModelAndView。

⑦HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。

⑧DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

⑨ViewReslover解析后返回具体View。

⑩DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。DispatcherServlet响应用户

### （三）实现步骤

#### 1.xml配置

##### 1.1 pom.xml中导入Spring和SpringMVC坐标

```xml
<!--Spring坐标-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<!--SpringMVC坐标-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
```

##### 1.2  web.xml配置SpringMVC的核心(前端)控制器

```xml
<!--前端控制器-->
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<!--映射地址-->
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

##### 1.3 配置内部资源视图解析器

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- webapp/jsp/success.jsp  -->
    <property name="prefix" value="/jsp/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
```

##### 1.4 mvc注解驱动

```xml
<!--mvc的注解驱动  ps：集成了Jackson-->
<mvc:annotation-driven/>
```

##### 1.5 开放静态资源访问

```xml
<!--开放资源的访问权限-->
<!--<mvc:resources mapping="/js/**" location="/js/"/>
    <mvc:resources mapping="/img/**" location="/img/"/>-->

<!--在mvc DispatcherServlet 找不到资源的情况下,启用(tomcat)默认的handler找资源-->
<mvc:default-servlet-handler/>
```

##### 1.6 其他

```xml
<!--文件上传解析器-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="defaultEncoding" value="UTF-8"></property>
    <property name="maxUploadSize" value="5242800"></property>
</bean>

<!--声明转换器-->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <list>
            <bean class="com.wyq.converter.DateConverter"></bean>
        </list>
    </property>
</bean>
```

```java
//注意:实现的是springframework 提供的接口Converter
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String s) {
        SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Date date = null;
        try {
            date = format.parse(s);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return  date;
    }
}
```

#### 2.注解

##### 2.1 @RequestMapping

作用：用于建立请求 URL 和处理请求方法之间的对应关系

位置：

​      类上，请求URL 的第一级访问目录。此处不写的话，就相当于应用的根目录

​      方法上，请求 URL 的第二级访问目录，与类上的使用@ReqquestMapping标注的一级目录一起组成访问虚拟路径

属性：

​      value：用于指定请求的URL。它和path属性的作用是一样的

​      method：用于指定请求的方式

​      params：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的key和value必须和配置的一模一样

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @RequestMapping(value = "/quick",method = RequestMethod.GET,params = {"username"})
    public String save(){
        System.out.println("controller save running.....");
        return  "success";
    }
}
```

##### 2.2 Restful风格

```java
@RequestMapping("/quick17/{name}") // Restful风格
@ResponseBody
public void save17(@PathVariable(value = "name") String username) {
    System.out.println("username = " + username);
}

```

### （四）SpringMVC请求与响应

#### 1.响应

##### 1.1 页面跳转

1. 直接返回字符串

   

2. 通过ModelAndView对象返回

   ```java
   @RequestMapping(value="/quick2")
   public ModelAndView save1111(ModelAndView modelAndView){
       /*
         Model:模型 作用封装数据
         View：视图 作用展示数据
        */
       //设置模型数据
       modelAndView.addObject("msg","hello");
       //设置视图名称
       modelAndView.setViewName("success");
       return modelAndView;
   }
   
   @RequestMapping(value="/quick4")
   public String save4(Model model){
       model.addAttribute("username","ruby");
       return "success";
   }
   
   ```

##### 1.2 回写数据

1. 直接返回字符串

   ```java
   @RequestMapping(value="/quick9")
   @ResponseBody
   public String save9() throws IOException {
       User user = new User();
       user.setUsername("lisi");
       user.setAge(30);
       //使用json的转换工具将对象转换成json格式字符串在返回
       ObjectMapper objectMapper = new ObjectMapper();
       String json = objectMapper.writeValueAsString(user);
       return json;
   }
   
   ```

   在spring-mvc.xml中设置了mvc的注解驱动( `<mvc:annotation-driven/>` )后，可以简化以上代码

   ```java
   @RequestMapping(value="/quick9")
   @ResponseBody
   public User save9() throws IOException {
       User user = new User();
       user.setUsername("lisi");
       user.setAge(30);
       //期望SpringMVC自动将User转换成json格式的字符串
       return User;
   }
   
   ```

2. 返回对象或者集合

   在 SpringMVC 的各个组件中，处理器映射器、处理器适配器、视图解析器称为 SpringMVC 的三大组件。

   在Spring-mvc.xml配置文件中使用`<mvc:annotation-driven />`替代注解处理器和适配器的配置。

   同时，默认底层就会集成jackson进行对象或集合的json格式字符串的转换

   返回对象集合和返回对象相同，会自动转换成json字符串

#### 2.请求

##### 2.1 可获得的请求参数类型

- 基本类型参数
- POJO类型参数
- 数组类型参数
- 集合类型参数

##### 2.2 POJO类型

Controller中的业务方法的POJO参数的属性名与请求参数的name一致，参数值会自动映射匹配。

##### 2.3 集合类型

获得集合参数时，要将集合参数包装到一个POJO中才可以。

```java
public class VO {
     private List<User> userList;
     //...省略以下
}

```

```html
<form action="${pageContext.request.contextPath}/user/quick14" method="post">
    <%--表明是第一个User对象的username age--%>
        <input type="text" name="userList[0].username"><br/>
        <input type="text" name="userList[0].age"><br/>
        <input type="text" name="userList[1].username"><br/>
        <input type="text" name="userList[1].age"><br/>
        <input type="submit" value="提交"
</form>

```

##### 2.4 静态资源访问开启

SpringMVC的前端控制器DispatcherServlet的url-pattern配置的是/,代表对所有的资源都进行过滤操作，当静态资源加载时（如jQuery），会出现访问不到的情况。

```xml
<!--开放资源的访问-->
	<!--指定放行的资源-->	
    <!--<mvc:resources mapping="/js/**" location="/js/"/>
    <mvc:resources mapping="/img/**" location="/img/"/>-->

<!--找不到的资源交给tomcat默认handler去找-->
<mvc:default-servlet-handler/>

```

##### 2.5 配置全局乱码过滤器

```xml
<!--配置全局过滤的filter-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```

##### 2.6 参数绑定注解@RequestParam

```java
public void save(@RequestParam(value="name",required=false,defaultValue ="li") String username) 

```

##### 2.7 Restful风格的参数的获取

Restful风格的请求是使用“url+请求方式”表示一次请求目的的，HTTP 协议里面四个表示操作方式的动词如下：

GET：用于获取资源

POST：用于新建资源

PUT：用于更新资源

DELETE：用于删除资源  

例如：

/user/1    GET ：       得到 id = 1 的 user

/user/1   DELETE：  删除 id = 1 的 user

/user/1    PUT：       更新 id = 1 的 user

/user       POST：      新增 user

##### 2.8 获得请求头信息

`@RequestHeader(value = "User-Agent",required = false)String user_agent`

`@CookieValue(value = "JSESSIONID") String jsessionId`

#### 3. 文件上传

##### 3.1 实现要求

文件上传客户端表单需要满足：

表单项type=“file”

表单的提交方式是post

表单的enctype属性是多部分表单形式，及enctype=“multipart/form-data”

##### 3.2 步骤

1. 添加依赖

   ```xml
   <dependency>
         <groupId>commons-fileupload</groupId>
         <artifactId>commons-fileupload</artifactId>
         <version>1.3.1</version>
   </dependency>
   <dependency>
         <groupId>commons-io</groupId>
         <artifactId>commons-io</artifactId>
         <version>2.3</version>
   </dependency>
   
   ```

2. 配置多媒体解析器

   ```xml
   <!--配置文件上传解析器-->
   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <property name="defaultEncoding" value="UTF-8"/>
       <property name="maxUploadSize" value="500000"/>
   </bean>
   
   ```

3. 前台

   ```xml
   <%--上传多个文件--%>
   <form action="${pageContext.request.contextPath}/user/quick21" method="post" enctype="multipart/form-data">
       姓名<input type="text" name="name"><br>
       文件1<input type="file" name="uploadFile"><br>
       文件2<input type="file" name="uploadFile"><br>
       <input type="submit" value="提交">
   </form>
   <%--单个文件--%>
   <form action="${pageContext.request.contextPath}/user/quick20" method="post" enctype="multipart/form-data">
       姓名<input type="text" name="name"><br>
       文件<input type="file" name="uploadFile"><br>
       <input type="submit" value="提交">
   </form>
   
   ```

4. 后台

   ```java
   @RequestMapping("/quick21")
   @ResponseBody
   public void save21(String name, MultipartFile[] uploadFile) throws IOException {
       System.out.println(name);
       for (MultipartFile multipartFile : uploadFile) {
           String originalFilename = multipartFile.getOriginalFilename();
           multipartFile.transferTo(new File("d:\\upload\\"+originalFilename));
       }
   }
   
   @RequestMapping("/quick20")
   @ResponseBody
   public void save20(String name, MultipartFile uploadFile) throws IOException {
       System.out.println(name);
       String originalFilename = uploadFile.getOriginalFilename();
       uploadFile.transferTo(new File("d:\\"+originalFilename));
   }
   
   ```

### （五）Interceptor拦截器

#### 1. 入门

##### 1.1作用

Spring MVC 的拦截器**类似**于 Servlet  开发中的过滤器 Filter，用于对处理器进行预处理和后处理。

将拦截器按一定的顺序联结成一条链，这条链称为拦截器链（InterceptorChain）。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。拦截器也是AOP思想的具体实现。

##### 1.2 interceptor和filter区别

| 区别     | interceptor（拦截器）                                    | filter（过滤器）                                    |
| -------- | -------------------------------------------------------- | --------------------------------------------------- |
| 使用范围 | 使用SpringMVC框架的工程                                  | JavaWeb工程                                         |
| 拦截范围 | 只会拦截访问控制器的方法，如果访问的是jsp,html等不会拦截 | 在url-pattern中配置了/*之后，会拦截所有要访问的资源 |

##### 1.3 实现步骤

1. 创建拦截器类实现HandlerInterceptor接口

   ```java
   public class MyInterceptor1 implements HandlerInterceptor {
       //在目标方法执行之前 执行
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
           System.out.println("preHandle.....");
           return true; //放行
   }
       //在目标方法执行之后 视图对象返回之前执行
       public void postHandle(){}
       //在流程都执行完毕后 执行
       public void afterCompletion(){}
   }
   
   
   ```

2. 配置拦截器

   ```xml
   <!--配置拦截器-->
       <mvc:interceptors>
          <mvc:interceptor>
               <mvc:mapping path="/**"/>
               <bean class="com.wyq.intercptor.MyInterceptor"></bean>
           </mvc:interceptor>
           <!--<mvc:interceptor>-->
               <!--<mvc:mapping path="/**"/>-->
               <!--<bean class="com.wyq.intercptor.MyInterceptor2"></bean>-->
           <!--</mvc:interceptor>-->      
       </mvc:interceptors>
   
   
   ```

3. 测试拦截器的拦截效果

   preHandle...
   preHandle222222222222...
   target........
   postHandle222222222222...
   postHandle...
   afterCompletion2222222222...
   afterCompletion...

##### 1.4 小结

当拦截器的preHandle方法返回true则会执行目标资源，如果返回false则不执行目标资源

多个拦截器情况下，配置在前的先执行，配置在后的后执行

拦截器中的方法执行顺序是：preHandler-------目标资源----postHandle---- afterCompletion

#### 2. 使用拦截器实现登录权限控制

##### 2.1 编写拦截器

```java
public class PrivilegeInterceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws IOException {
        //逻辑：判断用户是否登录  本质：判断session中有没有user
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        if(user==null){
            //没有登录
            response.sendRedirect(request.getContextPath()+"/login.jsp");
            return false;
        }
        //放行  访问目标资源
        return true;
    }
}

```

##### 2.2 配置拦截器

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <!--配置对哪些资源执行拦截操作-->
        <mvc:mapping path="/**"/>
        <!--配置哪些资源排除拦截操作-->
        <mvc:exclude-mapping path="/user/login"/>
        <bean class="com.itheima.interceptor.PrivilegeInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>

```

### （六）SpringMVC异常处理

#### 1.异常处理思路

系统中异常包括两类：预期异常和运行时异常RuntimeException，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试等手段减少运行时异常的发生。

系统的Dao、Service、Controller出现都通过throws Exception向上抛出，最后由SpringMVC前端控制器交由异常处理器进行异常处理。
![1565004037016](C:\Users\29789\AppData\Local\Temp\1565004037016.png)

#### 2.SpringMVC异常处理

##### 2.1 使用SpringMVC提供的简单异常处理器

```xml
<!--配置简单映射异常处理器-->
<bean class=“org.springframework.web.servlet.handler.SimpleMappingExceptionResolver”>    <property name=“defaultErrorView” value=“error”/>   默认错误视图
    <property name=“exceptionMappings”>
        <map>		异常类型		                             错误视图
            <entry key="com.itheima.exception.MyException" value="error"/>
            <entry key="java.lang.ClassCastException" value="error"/>
        </map>
    </property>
</bean>

```

##### 2.2 实现Spring的异常处理接口HandlerExceptionResolver 自定义自己的异常处理器

1. 创建异常处理器类实现HandlerExceptionResolver

   ```java
   public class MyExceptionResolver implements HandlerExceptionResolver {
   @Override
   public ModelAndView resolveException(HttpServletRequest request, 
       HttpServletResponse response, Object handler, Exception ex) {
       //处理异常的代码实现
       //创建ModelAndView对象
       ModelAndView modelAndView = new ModelAndView(); 
       modelAndView.setViewName("exceptionPage");
       return modelAndView;
       }
   }
   
   ```

2. 配置异常处理器

3. 编写异常页面

4. 测试异常跳转