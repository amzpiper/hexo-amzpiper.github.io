---
title: Spring-MVC笔记
date: 2021-08-13 10:00:41
comments: true
tags: 
    - Spring-MVC
categories: 后端
thumbnail: https://spring.io/images/spring-logo-9146a4d3298760c2e7e49595184e1975.svg
banner: https://spring.io/images/spring-logo-9146a4d3298760c2e7e49595184e1975.svg
cover: https://spring.io/images/spring-logo-9146a4d3298760c2e7e49595184e1975.svg
---

# 1、回顾Servlet

javaSE：

javaWeb：

SSM框架：**研究官方文档，锻炼自学能力，锻炼笔记能力，锻炼项目能力**。

SpringMVC+Vue+SpringBoot+SpringCloud+Linux

SSM=Javaweb做项目

Spring：IOC和AOP

SpringMVC：SpringMVC执行流程！

尽量跟着官方文档！

## 1.1、什么是MVC：

MVC是模型(Model)、视图(View)、控制器(Controller)的简写，是一种软件设计规范。

是将业务逻辑、数据、显示分离的方法来组织代码。

MVC主要作用是降低了视图与业务逻辑间的双向偶合。

MVC不是一种设计模式，MVC是一种架构模式。当然不同的MVC存在差异。

**重要概念：**

1、转发和重定向

2、前端 数据传输 实体类

实体类：用户名、密码、生日等20个

前端：用户名、密码

有必要创建bean来接收数据吗？会有好多null

pogo：User

vo：UserVo  还是实体类，拆掉了很多不必要的字段

dto：数据传输时对象

Model（模型）：数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据Dao） 和 服务层（行为Service）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。
View（视图）：负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。
Controller（控制器）：接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。 也就是说控制器做了个调度员的工作。
最典型的MVC就是JSP + servlet + javabean的模式。

![Untitled](Untitled.png)

## 1.2、Model1时代

![Untitled](Untitled%201.png)

## 1.3、Model2时代

![Untitled](Untitled%202.png)

![Untitled](Untitled%203.png)

![Untitled](Untitled%204.png)

更好的是方便团队开发！java越大约好维护

**面试问题**：你们的项目的架构，是设计好的，还是演进的？

演进的！

有本书叫：淘宝的10年革命！

- Alibaba PHP
- 随着用户变大 Java
- 王坚 去IOE  MySQL
- MySQL：Ali自己的MySQL-》aliSQL、AliRedis
- 所有的项目从All in one —> 微服务，不是一撮而就。

## 1.4、回顾Servlet的实现

1、新建Maven项目，导入依赖

```xml
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.10</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet/servlet-api -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet.jsp/jsp-api -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet.jsp.jstl/jstl -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

2、创建springmvc-01-servlet的普通maven项目添加AddFrameworkSupport

![Untitled](Untitled%205.png)

3、导入依赖

```xml
<!-- https://mvnrepository.com/artifact/javax.servlet/servlet-api -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet.jsp/jsp-api -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
```

4、编写一个Servlet类，用来处理用户请求

```xml
package com.kuang.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取前端参数
        String method = req.getParameter("method");
        if (method.equals("add")) {
            req.getSession().setAttribute("msg","执行了add方法");
        }
        if (method.equals("delete")) {
            req.getSession().setAttribute("msg","执行了delete方法");
        }
        //2.调用业务层

        //3.试图转发或重定向
        //转发-用的多
        req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req,resp);
        //重定向
        //resp.sendRedirect("");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}
```

5、配置servlet,web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.kuang.servlet.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
    
    <session-config>
        <session-timeout>1500</session-timeout>
    </session-config>
    
    <welcome-file-list>
        <welcome-file>from.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

6、创建from.jsp(web/)和test.jsp(web/WEB-INF/jsp)

```xml
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/1
  Time: 16:30
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>From</title>
</head>
<body>

<form method="post" action="/hello">
    <input type="text" name="method">
    <input type="submit">
</form>

</body>
</html>
```

```xml
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/1
  Time: 16:26
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Test</title>
</head>
<body>

${msg}

</body>
</html>
```

7、配置并启动tomcat，测试

[http://localhost:8080/hello?method=add](http://localhost:8080/hello?method=add)

![Untitled](Untitled%206.png)

MVVM：M、V、VM：ViewModel双向绑定。

# 2、SpringMvc

## 2.1、什么是SpringMvc

### 2.1.1、概述

SpringMvc是SpringFramework的一部分，是基于Java实现的MVC的轻量级的Web框架。

新文档官网：[https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)

老文档：[https://docs.spring.io/spring-framework/docs/4.3.24.RELEASE/spring-framework-reference/html/](https://docs.spring.io/spring-framework/docs/4.3.24.RELEASE/spring-framework-reference/html/)

[https://docs.spring.io/spring-framework/docs/4.3.24.RELEASE/spring-framework-reference/html/mvc.html#mvc-servlet](https://docs.spring.io/spring-framework/docs/4.3.24.RELEASE/spring-framework-reference/html/mvc.html#mvc-servlet)

我们为什么学习SpringMVC呢？

特点：

1.轻量级，简单易学

2.高效，基于请求与响应的MVC框架

3.与Spring兼容性好，无缝结合，可以将所有SpringMVC中用到的类，注册到Spring中！

4.约定优于配置

5.功能强大：RESTful、数据验证、格式化、本地化、主题等

6.简单灵活

### 2.2.2、搭建HelloMVC

**springmvc-02-hellomvc**

步骤：

1.新建moudle，添加web支持！

2.导入Springmvc依赖

3.配置web.xml，注册DispatcherServlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--1.注册DispatcherServlet-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

</web-app>
```

4.新建springmvc-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--1.处理器映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!--2.处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--3.视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

5.编写操作接口Controller

```xml
package com.kuang.controller;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
public class HelloController implements Controller {

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //ModelAndView 模型和试图
        ModelAndView mv = new ModelAndView();

        //封装对象，放到ModelAndView中
        mv.addObject("msg","HelloSpringMVC");
        //封装要跳转的试图，放到ModelAndView中,/WEB-INF/jsp/hello.jsp
        mv.setViewName("hello");
        return null;
    }

}
```

6.将Controller注册到bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--1.处理器映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!--2.处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--3.视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--HelloController-->
    <bean id="/hello" class="com.kuang.controller.HelloController"/>
</beans>
```

7.创建跳转页面/WEB-INF/jsp/hello.jsp

```xml
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/1
  Time: 19:27
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>${msg}</title>
</head>
<body>

${msg}

</body>
</html>
```

**注意问题：**

1.可能遇到404，问题，排查步骤：

1.查看控制台输出，缺少什么jar包

2.如果jar包存在，显示无法输出，就在IDEA的项目发布中，添加lib依赖！

3.重启Tomcat

![Untitled](Untitled%207.png)

4.D:\Application\apache-tomcat-9.0.34\apache-tomcat-9.0.34\conf

```xml
109   tomcat.util.scan.StandardJarScanFilter.jarsToSkip=***.jar**
```

5.检查是不是<servlet-mapping>

```xml
<servlet-name>dispatcher</servlet-name>
  <url-pattern>/*</url-pattern>
</servlet-mapping>
即路径写错，/后面不能加*，通常情况下都是
/即可，表示拦截所有请求。
```

### 2.2.3、SpringMVC执行原理

**初理解：**

Spring的web框架围绕DispatcherServlet【调度Servlet】设计。

DispatcherServlet是请求分发的作用。将请求分发到不同的处理器。从Spring2.5和Java5开启可以使用注解的controller声明方式开发。 

![Untitled](Untitled%208.png)

DispatcherServlet实际是一个Servlet，继承自HttpServlet基类。也有doGET和doPOST等。

![Untitled](Untitled%209.png)

SpringMVC原理如下图：

![Untitled](Untitled%2010.png)

**再理解：**

![Untitled](Untitled%2011.png)

注：实现都是springmvc实现的，我们只需要实现虚线的（3个） 

分析执行流程：

1.DispatcherServlet是前端控制器，是整个SpringMVC的控制中心。用户发送的请求，都会被DispathcerServlet接收请求并拦截请求。

```xml
<!--1.注册DispatcherServlet-->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<!--
    springmvc中 / 和 /*
    /:只匹配所有的请求，不会匹配jsp页面
    /*:匹配所有的请求，包括jsp页面
-->
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    **<url-pattern>/</url-pattern>**
</servlet-mapping>
```

- 我们假设请求的url为：[http://localhost:8080/springmvc/hello](http://localhost:8080/hello)
- 如上url拆分为3部分：
    - [http://localhost:8080](http://localhost:8080/hello)服务域名
    - [springmvc/](http://localhost:8080/hello)是部署在服务器上的web站点
    - hello是控制器
- 通过分析，如上url表示：请求位于服务器[localhost:8080](http://localhost:8080/hello)上的springmvc站点的hello控制器

2.HandlerMapping为处理器映射。由DispathcherServlet自行调用。

HandlerMapping根据请求的url调用HandlerExecution查找handler处理器（/hello）。

```xml
<!--1.处理器映射器-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
```

在执行一个HandlerExecution操作后，把结果（hello）又返回给DispathcherServlet。

3.HandlerExecution表示具体的Handler，其主要作用是根据url（/hello）查找控制器，如上url查到的控制器为hello。

4.HandlerExecution将解析后的信息传递给DispathcherServlet，如解析控制器映射等。

5.DispathcherServlet调用HandlerAdapter，HandlerAdapter表示处理器适配器（如查u盘后，usb找usb适配器）。其按照特定的规则去执行Handler。找具体的hello控制器的类。

```xml
<!--2.处理器适配器-->
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

6.HandlerAdapter让具体的控制器Controller（HelloController.java）执行。

7. Controller执行相关操作，并将执行后的信息返回给HandlerAdapter，如ModelAndView。

8.HandlerAdapter将Controller将执行后的信息ModelAndView（视图逻辑名和模型）返回给DispathcherServlet。

9.DispathcherServlet将调用视图解析器ViewResolver。

ViewResolver来解析DispathcherServlet传递过来的ModelAndView里的逻辑视图名，并拼接视图的名字，找到对应的视图。

将ModelAndView里的Model数据渲染到视图上。

```xml
<!--3.视图解析器：DispathcherServlet传给他ModelAndView
    1.获取了ModelAndView的数据
    2.解析ModelAndView的视图名字
    3.拼接视图的名字，找到对应的视图
    4.将ModelAndView里的Model数据渲染到视图上
-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

10.视图解析器ViewResolver将解析的视图名传递给DispathcherServlet。

11.DispathcherServlet根据视图解析器ViewResolver的结果调用具体的视图。

12.最终视图呈现给用户。

原理**总结：先拦截、再找url对应的controller、再找对应的视图、再展示视图。**

我们需要做的只有controller调用service层，然后返回视图名和数据。

**注意：springmvc中 / 和 /***
/ : 只匹配所有的请求，不会匹配jsp页面
/*: 匹配所有的请求，包括jsp页面

小结：实际开发不会这么写，用注解开发。

## 2.2、注解开发SpringMvc

**springmvc-03-annotation**

注：maven文件过滤问题

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

1、创建maven项目，引入依赖和引入web支持

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/junit/junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.10</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/javax.servlet/servlet-api -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/jsp-api -->
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.2</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/javax.servlet.jsp.jstl/jstl -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
```

2、配置web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
    <welcome-file-list>
        <welcome-file>/index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

3、添加spring配置文件

让IOC的注解生效

静态资源过滤：HTML、JS、CSS、图片、视频...

MVC的注解驱动(注册处理器映射器和处理器适配器的注解)

配置视图解析器

在resource目录添加spring-mvc.xml文件。为了支持注解的IOC，设置了自动扫描包功能。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <!--自动扫描包，让报下注解生效，由IOC统一管理-->
    <context:component-scan base-package="com.kuang"/>

    <!--让SpringMVC不出来静态资源 .css .js .html .mp3 .mp4等-->
    <mvc:default-servlet-handler />

    <!--
        支持mvc注解驱动
        在spring中一般采用@RequestMapping注解来完成映射关系
        想要@RequestMapping注解直接生效，必须向上下文注册DefaultAnnotationHandlerMapping和一个AnnotationMethodHandlerAdapter实例
        这两个实例分别在类级别和方法级别处理
        而annotation-driven配置帮助我们完成这2个实例配置注册IOC
        就不需要处理器映射器和处理器适配器了
    -->
    <mvc:annotation-driven />

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

在视图解析器中我们把视图文件放到/WEB-INF/jsp目录下，保证视图安全。

4、创建Controller

```xml
package com.kuang.comtroller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
@Controller
//@RestController 不调用视图解析器，直接返回数据，json格式用他
@RequestMapping("/hello")
public class HelloController {

    /**
     * 访问地址 http://localhost/hello/h1
     * 封装数据到 model
     * @param model
     */
    @RequestMapping(value ="/h1")
    public String hello(Model model) {
        //向模型中添加属性msg和值，可以在jsp中取出来并渲染
        model.addAttribute("msg", "Hello,SpringMVCAnnotation!");

        //会被视图解析器处理,hello是视图名字
        return "hello";
    }
}
```

5、创建jsp页面，/WEB-INF/jsp/hello.jsp

```xml
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/1
  Time: 22:11
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

${msg}

</body>
</html>
```

6.测试[http://localhost:8080/hello/h1](http://localhost:8080/hello/h1)

7.有问题，可以在项目结构中添加lib包

![Untitled](Untitled%2012.png)

8.步骤总结：

创建项目

导入jar包

添加web支持

在web.xml在配置dispatcherservlet

在resource下创建springmvc.xml

创建controller

## 2.3、Controller配置总结

### 2.3.1、控制器Controller

控制器提供访问程序的行为，通常通过实现接口定义或注解定义两种方式。

控制器负责解析用户请求并将器转换为一个模型，返回到视图解析器。

在SpringMVC中，一个控制器中可以包含多个方法。

在SpringMVC中，对于Controller的配置有很多种。

我们来看看有哪些方式可以实现：

### 2.3.2、使用实现Controller接口的方式

**springmvc-04-controller**

**测试：**

mvc配置文件只留下视图解析器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <!--自动扫描包，让报下注解生效，由IOC统一管理-->
    <!--<context:component-scan base-package="com.kuang"/>-->

    <!--让SpringMVC不出来静态资源 .css .js .html .mp3 .mp4等-->
    <!--<mvc:default-servlet-handler />-->

    <!--
        支持mvc注解驱动
        在spring中一般采用@RequestMapping注解来完成映射关系
        想要@RequestMapping注解直接生效，必须向上下文注册DefaultAnnotationHandlerMapping和一个AnnotationMethodHandlerAdapter实例
        这两个实例分别在类级别和方法级别处理
        而annotation-driven配置帮助我们完成这2个实例配置注册IOC
        就不需要处理器映射器和处理器适配器了
    -->
    <!--<mvc:annotation-driven />-->

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

1、编写一个Controller类，ControllerTest1

```xml
package com.kuang.controller;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
//只要实现了Controller接口的类，这就是一个控制器
public class ControllerTest1 implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv = new ModelAndView();

        mv.addObject("msg","ControllerTest1");
        mv.setViewName("test");
        return mv;
    }
}
```

2、在spring配置文件中添加ControllerTest1的bean

```xml
<bean name="/t1" class="com.kuang.controller.ControllerTest1"/>
```

说明：

这里没有显示的配置映射器和适配器。

一个控制器只能写一个方法。

### 2.3.3、使用注解实现Controller方式

回忆：IOC时还讲过和Controller效果一样的注解

```xml
@Component //只是组件意思
@Service  //业务层
@Controller //控制层
@Repository  //dao层
```

1、创建ControllerTest2

使用注解必须开启包扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <!--自动扫描包，让报下注解生效，由IOC统一管理-->
    <context:component-scan base-package="com.kuang"/>

    <!--让SpringMVC不出来静态资源 .css .js .html .mp3 .mp4等-->
    <!--<mvc:default-servlet-handler />-->

    <!--
        支持mvc注解驱动
        在spring中一般采用@RequestMapping注解来完成映射关系
        想要@RequestMapping注解直接生效，必须向上下文注册DefaultAnnotationHandlerMapping和一个AnnotationMethodHandlerAdapter实例
        这两个实例分别在类级别和方法级别处理
        而annotation-driven配置帮助我们完成这2个实例配置注册IOC
        就不需要处理器映射器和处理器适配器了
    -->
    <!--<mvc:annotation-driven />-->

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <bean name="/t1" class="com.kuang.controller.ControllerTest1"/>

</beans>
```

```xml
package com.kuang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
@Controller
public class ControllerTest2 {

    @RequestMapping("/t2")
    public String test1(Model model) {
        model.addAttribute("msg", "ControllerTest2");
        return "test";
    }
}
```

**说明：**

没有开启<!--<mvc:default-servlet-handler />-->、<!--<mvc:annotation-driven />-->，也可访问

[http://localhost:8080/t2](http://localhost:8080/t2)。

jsp可以复用。

除了这2中还有2种方式。最常用注解。

**idea使用技巧：**

改java代码使用redeploy，改配置文件需要重新启动，只改前端页面使用刷新。

## 2.4、RequestMappimg

@RequestMapping注解用于映射url到控制器或一个特定的方法。

1、创建ControllerTest3

```xml
package com.kuang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
@Controller
@RequestMapping("/c3")
public class ControllerTest3 {

    @RequestMapping("/t1")
    public String test1(Model model) {
        model.addAttribute("msg", "ControllerTest2");
        return "test";
    }
}
```

2、访问[http://localhost:8080/c3/t1](http://localhost:8080/c3/t1)

要先访问类，在访问方法。admin可以在类上放一个admin，其他都在一个类中写方法就好了。

## 2.5、RestFul风格

### 2.5.1、概念

RestFul就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这个风格设计的软件**更简洁、更有层次感、更易于实现缓存机制**。

### 2.5.2、功能

资源：互联网所有的事务都可以被抽象为资源

资源操作：使用POST、DELETE、PUT、GET，使用不同方法对资源进行操作

分别对应：添加、删除、修改、查询

### 2.5.3、传统方式操作资源

通过不同的参数和链接实现不同的效果！方法单一，只有post和get

[http://localhost/item/queryItem.action?id=1](http://localhost/item/queryItem.action?id=1)  查询，GET

[http://localhost/item/saveItem.action](http://localhost/item/queryItem.action?id=1)  新增，POST

[http://localhost/item/updateItem.action](http://localhost/item/queryItem.action?id=1)  更新，POST

[http://localhost/item/deleteItem.action?id=1](http://localhost/item/queryItem.action?id=1)  删除，GET或POST

### 2.5.4、RestFul风格操作资源

可以通过不同的请求方式(method)实现不同的效果！如下：请求地址一样，但是功能可以不一样!

[http://localhost/item/1](http://localhost/item/queryItem.action?id=1)  查询，GET

[http://localhost/item](http://localhost/item/queryItem.action?id=1)  新增，POST

[http://localhost/item](http://localhost/item/queryItem.action?id=1)  更新，PUT

[http://localhost/item/1](http://localhost/item/queryItem.action?id=1)  删除，DELETE

### 2.5.5、学习测试

**先用传统方式:**

1、在**springmvc-04-controller**中新建RestFulController

```xml
package com.kuang.controller;

import com.sun.org.apache.xpath.internal.operations.Mod;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
@Controller
public class RestFulController {

    @RequestMapping("/add")
    public String test1(int a, int b, Model model) {
        int res = a + b;
        model.addAttribute("msg", "结果为：" + res);
        return "test";
    }
}
```

2、访问:[http://localhost:8080/add?a=1&b=2](http://localhost:8080/add?a=1&b=2)

**修改用RestFul方式:**

在SpringMVC中可以使用@PathVariable注解，让方法的参数值绑定到一个URI模板变量中。

1、修改RestFulController

```xml
package com.kuang.controller;

import com.sun.org.apache.xpath.internal.operations.Mod;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
@Controller
public class RestFulController {

    //@RequestMapping(value="/add/{a}/{b}",method  = RequestMethod.GET)
    @GetMapping("/add/{a}/{b}")
    public String test1(@PathVariable int a, @PathVariable int b, Model model) {
        int res = a + b;
        model.addAttribute("msg", "结果为：" + res);
        return "test";
    }

}
```

2.测试访问[http://localhost:8080/add/1/2](http://localhost:8080/add/1/2) GET

方法基本的注解变体有如下几个：

```xml
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

3.相同路径不同效果，修改java

```xml
package com.kuang.controller;

import com.sun.org.apache.xpath.internal.operations.Mod;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

/**
 * @author 郭宇航
 * @date 2021/10/1
 * @apiNote
 */
@Controller
public class RestFulController {

    //@RequestMapping(value="/add/{a}/{b}",method  = RequestMethod.GET)
    @GetMapping("/add/{a}/{b}")
    public String test1(@PathVariable int a, @PathVariable int b, Model model) {
        int res = a + b;
        model.addAttribute("msg", "和为：" + res);
        return "test";
    }

    //@RequestMapping(value="/add/{a}/{b}",method  = RequestMethod.GET)
    @PostMapping("/add/{a}/{b}")
    public String test2(@PathVariable int a, @PathVariable int b, Model model) {
        int res = a * b;
        model.addAttribute("msg", "乘积为：" + res);
        return "test";
    }

}
```

4.测试访问 

[http://localhost:8080/add/3/2](http://localhost:8080/add/1/2) GET  和为：5

[http://localhost:8080/add/3/2](http://localhost:8080/add/1/2) POST  乘积为：6

**小结：**

@RequestMapping有name、value、path，name会报错

restful不爆露参数名

小黄鸭调试法：向自己解释一遍，读一遍代码。描述到一半可能找到错误了！

## 2.6、结果跳转方式-重定向和转发

### 2.6.1、ModelAndView

设置ModelAndView对象，根据view的名称，和视图解析器跳到指定的页面。

页面：{视图解析器前缀}+viewName+{视图解析器后缀}

```xml
<!--视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
    <!--前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>
</bean>
```

对应的controller类

```xml
@Override
public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
    ModelAndView mv = new ModelAndView();

    mv.addObject("msg","ControllerTest1");
    mv.setViewName("test");
    return mv;
}
```

这些是最老的办法。

### 2.6.2、调用ServletApi

**在springmvc-04-controller中新建ModelTest**

通过设置ServletAPI，不需要视图解析器。

1.通过HttpServletResponse进行传输

2.通过HttpServletResponse实现重定向

3.HttpServletRequest实现转发

**新建ModelTest:**

```xml
@GetMapping("/m1")
public void m1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    response.getWriter().println("Hello,SpringMVC");
}

@GetMapping("/m2")
public void m2(HttpServletRequest request, HttpServletResponse response) throws IOException {
    //重定向
    response.sendRedirect("/index.jsp");
}

@GetMapping("/m3")
public void m3(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
    //转发
    request.setAttribute("msg","request.getRequestDispatcher");
    request.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(request, response);
}
```

不建议这样做

### 2.6.3、通过SpringMVC实现转发和重定向

**一、无视图解析器:**

需要自己拼接url

```xml
//springmvc-无视图解析器
http://localhost:8080/mvcm1
@GetMapping("/mvcm1")
public String mvcm1(){
    //转发
    return "/index.jsp";
}

http://localhost:8080/mvcm2
@GetMapping("/mvcm2")
public String mvcm2() {
    //转发
    return "forward:/index.jsp";
}

http://localhost:8080/mvcm3
@GetMapping("/mvcm3")
public String mvcm3() {
    //重定向
    return "redirect:/index.jsp";
}
```

转发和重定向都需要自己拼接完整的路径。

返回视图名中添加了前缀forward:或redirect:后不会走视图解析器。

**二、配置视图解析器:**

```xml
//Springmvc-配置视图解析器
http://localhost:8080/mvcm4
@GetMapping("/mvcm4")
public String mvcm4(){
    //转发
    return "test";
}

http://localhost:8080/mvcm5
@GetMapping("/mvcm5")
public String mvcm5() {
    //重定向
    return "redirect:/index.jsp";
}
```

直接返回视图名会走视图解析器。转发不需要拼接完整路径。

添加了前缀redirect:后不会走视图解析器。需要自己拼接完整路径。

## 2.7、处理提交数据和数据返回前端

**在springmvc-04-controller中新建ModelTest**

### 2.7.1、提交的字段名称和处理方法中的参数名一致

提交数据：GET [http://localhost:8080/user/test1?name=qinjiang](http://localhost:8080/user/test1?name=qinjiang)

处理方法：

```xml
@GetMapping("/test1")
public String test1(String name, Model model) {
    //1.接收前端参数
    System.out.println(name);
    //2.将返回结果传递给前端
    model.addAttribute("msg", name);
    //3.跳转试图
    return "test";
}
```

### 2.7.2、提交的字段名称和处理方法中的参数名不一致

@RequestParam("name") String userName

提交数据：GET [http://localhost:8080/user/test2?username=qinjiang](http://localhost:8080/user/test2?username=qinjiang)

处理方法：

```xml
@GetMapping("/test2")
public String test2(@RequestParam("username")String name, Model model) {
    //1.接收前端参数
    System.out.println(name);
    //2.将返回结果传递给前端
    model.addAttribute("msg", name);
    //3.跳转试图
    return "test";
}
```

**必须要写上RequestParam**

### 2.7.3、提交的一个对象

提交数据：GET [http://localhost:8080/user/test3?id=1&name=qinjiang&age=21](http://localhost:8080/user/test3?id=1&name=qinjiang&age=21)

处理方法：

```xml
/**
 * 前端传递对象：id，name，age
 * 接收参数时，判断参数的名字，假设名字直接在方法参数里，直接使用
 * 假设传递的是一个对象User，匹配User对象中的字段名。匹配不到为null
 */
@GetMapping("/test3")
public String test3(User user, Model model) {
    //1.接收前端参数
    System.out.println(user.toString());
    //2.将返回结果传递给前端
    model.addAttribute("msg", user.toString());
    //3.跳转试图
    return "test";
}
```

### 2.7.4、通过ModelAndView返回数据

```xml
@Override
public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
    ModelAndView mv = new ModelAndView();

    mv.addObject("msg","ControllerTest1");
    mv.setViewName("test");
    return mv;
}
```

### 2.7.5、通过ModelMap返回数据

class ModelMap extends LinkedHashMap<String, Object>

ModelMap:继承了LinkedHashMap，继承了LinkedHashMap的所有方法和特性。

Model :精简版，大部分使用它，只有简单几个方法，简化了操作和理解。

提交数据：GET [http://localhost:8080/user/test4?id=1&name=qinjiang&age=21](http://localhost:8080/user/test3?id=1&name=qinjiang&age=21)

处理方法：

```xml
@GetMapping("/test4")
public String test4(User user,ModelMap map) {
    //1.接收前端参数
    System.out.println(user.toString());
    //2.将返回结果传递给前端
    map.addAttribute("msg", user.toString());
    //3.跳转试图
    return "test";
}
```

### 2.7.6、通过Model返回数据

```xml
@RequestMapping("/t2")
public String test1(Model model) {
    model.addAttribute("msg", "ControllerTest2");
    return "test";
}
```

## 2.8、乱码问题

### 2.8.1、测试

**在springmvc-04-controller中新建EncodingController**

```xml
package com.kuang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PostMapping;

/**
 * @author 郭宇航
 * @date 2021/10/2
 * @apiNote
 */
@Controller
public class EncodingController {

    @PostMapping("/e1/t1")
    public String test1(String name, Model model) {
        System.out.println(name);
        model.addAttribute("msg", name);
        return "test";
    }

}
```

form.jsp

```xml
<form method="post" action="/e1/t1">
  <input type="text" name="name">
  <input type="submit">
</form>
```

测试结果为：é­å®èª

### 2.8.2、过滤器解决乱码

web.xml，配置springmvc的乱码过滤器，注意url-pattern配置为/*

```xml
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

**注意**：/只过滤请求，如：POST [http://localhost:8080/e1/t1?name=秦将](http://localhost:8080/e1/t1?name=%E7%A7%A6%E5%B0%86)，

jsp页面提交的会依旧乱码。

**乱码解决了，如果解决不了处理思路：**

1.检查tomcat配置文件设置编码：

```xml
<Connector URLEncoding="utf-8" port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

## 2.9、JSON

### 2.9.1、什么是JSON

- JSON（JavaScript Object Notation，JS对象标记）是一种轻量级的数据交换格式，目前使用最广泛。
- 采用了完全独立于编程语言的纯文本格式来储存和表示数据。
- 简介和清晰的层次结构使得JSON成为理想的数据交换语言。
- 易于人阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输率。

在JavaScript中，一切皆对象。因此，任何JavaScript支持的类型都可以通过JSON来表示，字符串、数字、对象、数组等。语法格式：

对象表示为键值对，数据由逗号分隔。

花括号保存对象。

方括号保存数组。

**JSON键值对**是用来保存对象的一种方式。键/值对组合中的键名写在前并用双引号包裹，使用冒号分割，然后是值：

```xml
{"name":"qinjiang"}
```

**JSON和JavaScript对象的转换**

从JSON字符串转换为JavaScript对象，使用JSON.parse()方法：

```xml
var obj = JSON.parse('{"a":"Hello", "b":"world"}');
//结果是 {a:'Hello', b:'world'}
```

从JavaScript对象转换为JSON字符串，使用JSON.stringify()方法：

```xml
var json= JSON.stringify({a:'Hello', b:'world'});
//结果是 '{"a":"Hello", "b":"world"}'
```

**代码测试:**

**springmvc-05-json**

```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JSONTEST</title>

    <script type="application/javascript">
        //编写一个对象
        var user = {
            name : "秦将",
            age : 3,
            sex : "男"
        }
        console.log(user);

        //将js对象转换为JSON对象
        // var json = JSON.stringify({a:'Hello', b:'world'});
        var json = JSON.stringify(user);
        console.log(json);

        //将JSON对象转换为js对象
        var obj = JSON.parse(json);
        console.log(obj);
    </script>

</head>
<body>

</body>
</html>
```

### 2.9.2、Controller返回JSON数据-Jackson

**springmvc-05-json**

Jackson是目前比较好用的json解析工具。

阿里巴巴的fastjson。

1.导入依赖:

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

2.web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--springmav解决乱码的过滤器-->
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <welcome-file-list>
        <welcome-file>/index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

3.springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <!--自动扫描包，让报下注解生效，由IOC统一管理-->
    <context:component-scan base-package="com.kuang"/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--JSON数据乱码统一解决-->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"/>
            </bean>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                        <property name="failOnEmptyBeans" value="false"/>
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

</beans>
```

4.新建User实体类

```xml
package com.kuang.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * @author 郭宇航
 * @date 2021/10/2
 * @apiNote
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String name;
    private int age;
    private String sex;
}
```

5.引入注解，新建UserController：

- @ResponseBody //不走视图解析器,直接返回字符串
- @RequestMapping(value = "/j1", produces = "application/json;charset=utf-8"中的produces 定义返回格式，开发不会这么做，springmvc有个全能的。
- @RestController  所有的方法都不在视图解析器,直接返回数据

**ObjectMapper把普通对象转换为json字符串**

```xml
package com.kuang.controller;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.kuang.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 郭宇航
 * @date 2021/10/2
 * @apiNote
 */
//@Controller
@RestController //所有的方法都不在视图解析器,直接返回数据
public class UserController {
	
    @RequestMapping(value = "/j1")
    //@ResponseBody //不走视图解析器,直接返回字符串
    public String json1() throws JsonProcessingException {
        User user = new User("亲家境",12,"男");

        //jackson,ObjectMapper
        //ObjectMapper把对象转换为json字符串
        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(user);
        return json;
    }
}
```

6.测试访问 [http://localhost:8080/j1](http://localhost:8080/j1)

```xml
{
	"name": "亲家境",
	"age": 12,
	"sex": "男"
}
```

**7.集合转json字符串**

```java
/**
 * 集合对象
 * @return
 * @throws JsonProcessingException
 */
@RequestMapping(value = "/j2")
public String json2() throws JsonProcessingException {
    List<User> users = new ArrayList<>();
    User user = new User("亲家境", 12, "男");
    User user2 = new User("亲家境", 12, "男");
    User user3 = new User("亲家境", 12, "男");
    User user4 = new User("亲家境", 12, "男");
    users.add(user);
    users.add(user2);
    users.add(user3);
    users.add(user4);

    //ObjectMapper把对象转换为json字符串
    ObjectMapper mapper = new ObjectMapper();
    String json = mapper.writeValueAsString(users);

    return json;
}
```

8.测试访问 [http://localhost:8080/j](http://localhost:8080/j1)2

```json
[
	{
		"name": "亲家境",
		"age": 12,
		"sex": "男"
	},
	{
		"name": "亲家境",
		"age": 12,
		"sex": "男"
	},
	{
		"name": "亲家境",
		"age": 12,
		"sex": "男"
	},
	{
		"name": "亲家境",
		"age": 12,
		"sex": "男"
	}
]
```

9**.时间地下转json字符串**

```java
/**
 * 时间对象
 * @return
 * @throws JsonProcessingException
 */
@RequestMapping(value = "/j3")
public String json3() throws JsonProcessingException {
    //ObjectMapper把对象转换为json字符串
    ObjectMapper mapper = new ObjectMapper();
    Date date = new Date();
    //将时间解析后的默认格式为Timestamp 1633167988542
    String json = mapper.writeValueAsString(date);

    //第一种格式化时间:手动
    //自定义返回格式 yyy-MM-dd HH:mm:ss
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyy-MM-dd HH:mm:ss");
    json = mapper.writeValueAsString(simpleDateFormat.format(date));

    //第二种：使用mapper.configure配置
    //关闭使用时间戳的方式
    mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
    //自定义日期格式
    SimpleDateFormat sdf = new SimpleDateFormat("yyy-MM-dd HH:mm:ss");
    //设置为自定义格式
    mapper.setDateFormat(sdf);

    json = mapper.writeValueAsString(simpleDateFormat.format(date));
    return json;
}
```

10.测试访问 [http://localhost:8080/j](http://localhost:8080/j1)3

11.提取公共的操作ObjectMapper的JsonUtils 类

```java
package com.kuang.utils;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.text.SimpleDateFormat;

/**
 * @author 郭宇航
 * @date 2021/10/2
 * @apiNote
 */
public class JsonUtils {

    /**
     * 转object为json字符串，和配置时间返回格式
     * @param object 转json字符串的对象
     * @return json字符串
     */
    public static String getJson(Object object) {
        ObjectMapper mapper = new ObjectMapper();
        try {
            return mapper.writeValueAsString(object);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 转object为json字符串，和配置时间返回格式
     * @param object 转json字符串的对象
     * @param sdf 时间返回格式字符串
     * @return json字符串
     */
    public static String getJson(Object object,String sdf) {
        ObjectMapper mapper = new ObjectMapper();

        //关闭使用时间戳的方式
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        //自定义日期格式
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat(sdf);
        //设置为自定义格式
        mapper.setDateFormat(simpleDateFormat);

        try {
            return mapper.writeValueAsString(object);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return null;
    }

}
```

12.测试

```java
/**
 * 测试使用jsonUtils
 * @return
 * @throws JsonProcessingException
 */
@RequestMapping(value = "/j4")
public String json4() throws JsonProcessingException {
    Date date = new Date();
    return JsonUtils.getJson(date,"yyy-MM-dd HH:mm:ss");
}
```

### 2.9.3、JSON数据乱码统一解决

@RequestMapping(value = "/j1", produces = "application/json;charset=utf-8"中的produces 定义返回格式，开发不会这么做，springmvc有个全能的。

可以在springmvc的配置文件中添加一段消息StringHttpMessageConverter转换配置！

```xml
<!--JSON数据乱码统一解决-->
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="UTF-8"/>
        </bean>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                    <property name="failOnEmptyBeans" value="false"/>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

### 2.9.4、Controller返回JSON数据-FastJson

fastjson.jar是阿里巴巴开发的专门用于java的包，方便json对象与javabean之间的转换。

依赖:

```java
<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.78</version>
</dependency>
```

**fastjson主要的三个类：**

- JSONObject代表json对象
    - JSONObject实现类Map接口
    - 对应json对象，通过各种形式的get()方法可以获得json对象的数据，也可以利用size()、isEmpty()等方法获得键值对的个数和是否为空。本质是通过Map的方法实现的。
- JSONArray代表json对象数组
    - 内部是由List接口的方法来完成的
- JSON代表JSONObject和JSONArray的转化
    - JSON类原发分析与使用
    - 仔细观察这些方法，主要是实现json对象、json对象数组、javabean对象、json字符串之间的相互转化。

**代码测试：**

```java
/**
 * 测试使用fastjson
 *
 * @return
 * @throws JsonProcessingException
 */
@RequestMapping(value = "/j5")
public String json5() throws JsonProcessingException {
    String json;

    //1.1java的list对象转json数组字符串
    List<User> users = new ArrayList<>();
    User user = new User("亲家境", 12, "男");
    User user2 = new User("亲家境", 12, "男");
    User user3 = new User("亲家境", 12, "男");
    User user4 = new User("亲家境", 12, "男");
    users.add(user);
    users.add(user2);
    users.add(user3);
    users.add(user4);
    json = JSON.toJSONString(users);
    System.out.println("java的list对象转json数组字符串:");
    System.out.println(json);

    //1.2json数组字符串转java的list对象
    List<User> jsonArray = JSON.parseArray(json,User.class);
    System.out.println("json数组字符串转java的list对象:");
    for (User user1 : jsonArray) {
        System.out.print(user1.toString());
    }

    //2.1java对象转json字符串
    json = JSON.toJSONString(user);
    System.out.println("java对象转json字符串:");
    System.out.println(json);

    //2.2json对象字符串转java对象
    user = JSON.parseObject(json, User.class);
    System.out.println("json对象字符串转java对象:");
    System.out.println(json);

    return json;
}
```

结果：

```java
java的list对象转json数组字符串:
[{"age":12,"name":"亲家境","sex":"男"},{"age":12,"name":"亲家境","sex":"男"},{"age":12,"name":"亲家境","sex":"男"},{"age":12,"name":"亲家境","sex":"男"}]
json数组字符串转java的list对象:
User(name=亲家境, age=12, sex=男)User(name=亲家境, age=12, sex=男)User(name=亲家境, age=12, sex=男)User(name=亲家境, age=12, sex=男)
java对象转json字符串:
{"age":12,"name":"亲家境","sex":"男"}
json对象字符串转java对象:
{"age":12,"name":"亲家境","sex":"男"}
```

## 2.10、SSM整合

需要熟练掌握MySQL数据库、Spring、JavaWeb及Mybatis。

**工具环境：**

- IDEA
- mybatis5.7.19
- tomcat9
- maven3.6

**数据库环境：**

```java
drop database if exists ssmbuild;
create database ssmbuild;
use ssmbuild;

drop table if exists books;
create table books(
	bookId int(10) not null auto_increment comment '书id',
    bookName varchar(100) not null comment '书名',
    bookCounts int(100) not null comment '数量',
    detail varchar(200) not null comment '描述',
    key bookId ( bookId )
) engine=InnoDB default charset=utf8;
insert into books(bookId,bookName,bookCounts,detail) values
(1,'Java',1,'从入门到精通'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',15,'从入门到进牢');
```

1、创建maven项目ssmbuild

2、导入依赖

<!--依赖:junit.数据库驱动.连接池.servlet.jsp.mybatis.mybatis-spring.spring-→

spring5.0以上使用mybatis2.*.*

```java
<!--依赖:junit.数据库驱动.连接池.servlet.jsp.mybatis.mybatis-spring.spring-->
<!--Junit-->
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

<!--数据库驱动-->
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.49</version>
</dependency>

<!--C3P0-->
<!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.5</version>
</dependency>

<!--Servlet - JSP-->
<!-- https://mvnrepository.com/artifact/javax.servlet/servlet-api -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet.jsp/jsp-api -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet.jsp.jstl/jstl -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>

<!--Mybatis-->
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
</dependency>

<!--SpringMVC-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.10</version>
</dependency>
<!--Spring JDBC-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.10</version>
</dependency>
<!--Spring Aop-->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.7</version>
</dependency>
```

3.maven过滤

```java
<!--静态资源导出问题-->
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

4.新建基本包结构

- **pojo**
- **dao**
- **service**
- **controller**
- filter
- utils

5.resources目录新建mybatis-config.xml、applicationContext.xml

mybatis-config.xml:

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

</configuration>
```

applicationContext.xml:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

</beans>
```

### 2.10.1、Mybatis层

1.db.properties

```java
driver=com.mysql.jdbc.Driver
#如果使用mysql8.0+，增加一个时区配置&serverTimezone=Asia/Shanghai
url=jdbc:mysql://39.106.63.189:3307/ssmbuild?useSSL=false&useUnicode=true&characterEncoding=utf8
username=root
password=root
```

2.编写mybatis-config.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--配置数据源，交给spring去做-->

    <!--1.日志-->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <!--2.取别名-->
    <typeAliases>
        <package name="com.kuang.pojo"/>
    </typeAliases>

    <!--3.Mapper文件注册-->
    <mappers>
        <mapper class="com.kuang.dao.BookMapper"/>
    </mappers>

</configuration>
```

3.pojo包下新建实体类Books

```java
package com.kuang.pojo;

import java.util.Objects;

/**
 * @author 郭宇航
 * @date 2021/10/3
 * @apiNote
 */
public class Books {

    private int bookId;
    private String bookName;
    private String bookCounts;
    private String detail;

    public Books() {
    }

    public Books(int bookId, String bookName, String bookCounts, String detail) {
        this.bookId = bookId;
        this.bookName = bookName;
        this.bookCounts = bookCounts;
        this.detail = detail;
    }

    public int getBookId() {
        return bookId;
    }

    public void setBookId(int bookId) {
        this.bookId = bookId;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public String getBookCounts() {
        return bookCounts;
    }

    public void setBookCounts(String bookCounts) {
        this.bookCounts = bookCounts;
    }

    public String getDetail() {
        return detail;
    }

    public void setDetail(String detail) {
        this.detail = detail;
    }

    @Override
    public String toString() {
        return "Books{" +
                "bookId=" + bookId +
                ", bookName='" + bookName + '\'' +
                ", bookCounts='" + bookCounts + '\'' +
                ", detail='" + detail + '\'' +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Books books = (Books) o;
        return bookId == books.bookId &&
                Objects.equals(bookName, books.bookName) &&
                Objects.equals(bookCounts, books.bookCounts) &&
                Objects.equals(detail, books.detail);
    }

    @Override
    public int hashCode() {
        return Objects.hash(bookId, bookName, bookCounts, detail);
    }
}
```

3.dao包下新建接口BookMapper

```java
package com.kuang.dao;

import com.kuang.pojo.Books;

import java.util.List;

/**
 * @author 郭宇航
 * @date 2021/10/3
 * @apiNote
 */
public interface BookMapper {

    /**
     * 增加一本书
     * @param books
     * @return
     */
    int addBook(Books books);

    /**
     * 删除一本书
     * @param id
     * @return
     */
    int deleteBook(int id);

    /**
     * 更新一本书
     * @param books
     * @return
     */
    int updateBook(Books books);

    /**
     * 查询一本书
     * @param id
     * @return
     */
    Books queryBookById(int id);

    /**
     * 查询所有书
     * @return
     */
    List<Books> queryAllBook();

}
```

4.dao包下新建接口BookMapper.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.dao.BookMapper">

    <insert id="addBook" parameterType="books">
        insert into books(bookName, bookCounts, detail)
        values (#{bookName},#{bookCounts},#{detail})
    </insert>

    <delete id="deleteBookById" parameterType="int">
        delete from books where bookId = #{bookId}
    </delete>

    <update id="updateBook" parameterType="books">
        update books
        set bookName = #{bookName}, bookCounts = #{bookCounts}, detail = #{detail}
        where bookId = #{bookId}
    </update>

    <select id="queryBookById" parameterType="int" resultType="books">
        select bookId,bookName,bookCounts,detail
        from books
        where bookId = #{bookId}
    </select>

    <select id="queryAllBook" resultType="books">
        select bookId,bookName,bookCounts,detail
        from books
    </select>
</mapper>
```

5.在mybatis-config.xml中注册Mapper.xml

```java
<mappers>
    <mapper class="com.kuang.dao.BookMapper"/>
</mappers>
```

6.service新建接口BookService

```java
package com.kuang.service;

import com.kuang.pojo.Books;
import java.util.List;

/**
 * @author 郭宇航
 * @date 2021/10/3
 * @apiNote
 */
public interface BookService {

    /**
     * 增加一本书
     *
     * @param books
     * @return
     */
    int addBook(Books books);

    /**
     * 删除一本书
     *
     * @param id
     * @return
     */
    int deleteBookById(int id);

    /**
     * 更新一本书
     *
     * @param books
     * @return
     */
    int updateBook(Books books);

    /**
     * 查询一本书
     *
     * @param id
     * @return
     */
    Books queryBookById(int id);

    /**
     * 查询所有书
     *
     * @return
     */
    List<Books> queryAllBook();
}
```

7.service新建接口实现类BookServiceImpl

```java
package com.kuang.service;

import com.kuang.dao.BookMapper;
import com.kuang.pojo.Books;

import java.util.List;

/**
 * @author 郭宇航
 * @date 2021/10/3
 * @apiNote
 */
public class BookServiceImpl implements BookService{

    //业务层调dao层：组合dao层
    private BookMapper bookMapper;
    public void setBookMapper(BookMapper bookMapper) {
        this.bookMapper = bookMapper;
    }

    @Override
    public int addBook(Books books) {
        return bookMapper.addBook(books);
    }

    @Override
    public int deleteBookById(int id) {
        return bookMapper.deleteBookById(id);
    }

    @Override
    public int updateBook(Books books) {
        return bookMapper.updateBook(books);
    }

    @Override
    public Books queryBookById(int id) {
        return bookMapper.queryBookById(id);
    }

    @Override
    public List<Books> queryAllBook() {
        return bookMapper.queryAllBook();
    }
}
```

业务层调dao层：需要组合dao层的mapper

还有事务等，直接springaop横切就行。

### 2.10.2、Spring层

1.在resources包下创建spring-dao.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--1.关联数据库文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--
    2.连接池:
        DriverManagerDataSource
        dbcp 半自动化操作，不自动连接
        c3p0 自动化的加载配置文件，自动设置到对象中！
        druid 很强大,公司用的多
        hikari springboot2.0默认集成
    -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUrl" value="jdbc:mysql://39.106.63.189:3307/ssmbuild?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8" />
        <property name="user" value="root" />
        <property name="password" value="root" />
        <!--c3p0:私有属性-->
        <property name="maxPoolSize" value="30" />
        <property name="minPoolSize" value="10" />
        <!--c3p0:关闭连接后不自动commit-->
        <property name="autoCommitOnClose" value="false" />
        <!--c3p0:获取连接超时时间-->
        <property name="checkoutTimeout" value="10000" />
        <!--c3p0:当前获取连接失败重试次数-->
        <property name="acquireRetryAttempts" value="2" />
    </bean>

    <!--3.整合sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--绑定连接池-->
        <property name="dataSource" ref="dataSource"/>
        <!--绑定Mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>

    <!--4.配置dao接口扫描包，动态的把Dao接口注入到Spring容器中.
    以前是写BookMapperImpl extends SqlSessionDaoSupport，使用sqlSessionTemplate。
    现在通过MapperScannerConfigurer动态实现了BookMapperImpl
    -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--注入sqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!--扫描的包-->
        <property name="basePackage" value="com.kuang.dao"/>
    </bean>
</beans>
```

2.在resources包下创建spring-service.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--1.扫描service下的包-->
    <context:component-scan base-package="com.kuang.service" />

    <!--2.将我们的所有业务类，注入到spring，通过配置或注解,这里使用配置-->
    <bean id="BookServiceImpl" class="com.kuang.service.BookServiceImpl">
        <property name="bookMapper" ref="bookMapper"/>
    </bean>

    <!--3.声明式事务，默认都创建事务-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--4.aop事务支持-自定义!-->
    <!--配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--给哪些方法配置事务-->
        <!--配置事务的传播特性-->
        <tx:attributes>
            <tx:method name="add" propagation="REQUIRED"/>
            <tx:method name="delete" propagation="REQUIRED"/>
            <tx:method name="update" propagation="REQUIRED"/>
            <tx:method name="query" propagation="REQUIRED" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <!--配置事务切入点-->
    <aop:config>
        <aop:pointcut id="txPointCut" expression="execution(* com.kuang.dao.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
    </aop:config>
</beans>
```

3.修改applicationContext.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--整合配置文件-->
    <import resource="classpath:spring-dao.xml"/>
    <import resource="classpath:spring-service.xml"/>

</beans>
```

### 2.10.3、SpringMVC层

1.增加web支持

2.配置webxml

dispathcerServlet、乱码处理等

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--1.DispatcherServlet-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--2.乱码过滤-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--3.默认Session过期时间 15分钟-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
</web-app>
```

3.在resource包下新建spring-mvc.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/cache/spring-mvc.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--1.注解驱动-->
    <mvc:annotation-driven />

    <!--2.静态资源过滤-->
    <mvc:default-servlet-handler />

    <!--3.扫描包controller-->
    <context:component-scan base-package="com.kuang.controller"/>

    <!--4.视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--前缀:WEB-INF下的jsp包-->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

4.在applicationContext.xml添加spring-mvc.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--整合配置文件-->
    <import resource="classpath:spring-dao.xml"/>
    <import resource="classpath:spring-service.xml"/>
    <import resource="classpath:spring-mvc.xml"/>

</beans>
```

### 2.10.4、查阅书籍

```
问题：bean不存在
步骤：
1.查看这个bean是否注入成功！
2.Junit单元测试,看我们的代码是否能查询处理结果
3.问题，db配置文件编码格式问题
4.springmvc整合的时候没调用到我们的service层的bean
    1.applicationContext没有注册bean x
    2.web.xml中，绑定配置文件出问题，应绑定applicationContext.xml而不是spring-mvc.xml
```

1.新建BookController，写listBook

```java
package com.kuang.controller;

import com.kuang.pojo.Books;
import com.kuang.service.BookService;
import com.kuang.service.BookServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.List;

/**
 * @author 郭宇航
 * @date 2021/10/3
 * @apiNote
 */
@Controller
public class BookController {

    @Autowired
    @Qualifier("BookServiceImpl")
    private BookService bookService;

    /**
     * 查询全部的书籍，并返回到一个书籍展示页面
     * @param model
     * @return 
     */
    @GetMapping("/book/allBook")
    public String listBook(Model model) {
        List<Books> list = bookService.queryAllBook();
        for (Books books : list) {
            System.out.println(books);
        }

        model.addAttribute("msg", list);

        return "allBook";
    }

}
```

2.在WEB-INF/jsp/下新建allBook.jsp

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/3
  Time: 16:04
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>书记展示</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

    <div class="row clearfix">
        <div class="col-md-12 column">
            <div class="page-header">
                <h1>
                    <small>书籍列表</small>
                </h1>
            </div>
        </div>
    </div>
    <a href="/book/toAddBookPiper">添加数据</a>

    <div class="row clearfix">
        <div class="col-md-12 column">
            <table class="table table-hover table-striped">
                <thead></thead>
                    <tr>书籍编号</tr>
                    <tr>书籍名称</tr>
                    <tr>书籍数量</tr>
                    <tr>书籍详情</tr>
                <tbody>
                    <c:forEach items="${bookList}" var="book">
                        <tr>
                            <td>${book.bookId}</td>
                            <td>${book.bookName}</td>
                            <td>${book.bookCounts}</td>
                            <td>${book.detail}</td>
                        </tr>
                    </c:forEach>
                </tbody>
            </table>
        </div>
    </div>

</body>
</html>
```

3.访问[http://localhost:8080/book/allBook](http://localhost:8080/book/allBook)

4.web.xml

```java
<jsp-config>
    <taglib>
        <taglib-uri>http://java.sun.com/jsp/jstl/core</taglib-uri>
        <taglib-location>/WEB-INF/c.tld</taglib-location>
    </taglib>
</jsp-config>
```

### 2.10.4、添加书籍

1.allBook.jsp界面增加跳转增加书籍界面功能

```java
<a href="/book/toAddBookPiper">添加数据</a>
```

2.写toAddBookPiper方法

```java
/**
 * 跳转addBook.jsp
 * @return
 */
@GetMapping("/book/toAddBookPiper")
public String toAddBookPiper() {
    return "addBook";
}
```

3.增加书籍页面addBook.jsp

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/3
  Time: 16:04
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>书记展示</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

    <div class="row clearfix">
        <div class="col-md-12 column">
            <div class="page-header">
                <h1>
                    <small>书籍列表</small>
                </h1>
            </div>
        </div>
    </div>
    <a href="/book/toAddBookPiper">添加数据</a>

    <div class="row clearfix">
        <div class="col-md-12 column">
            <table class="table table-hover table-striped">
                <thead>
                    <tr>书籍编号</tr>
                    <tr>书籍名称</tr>
                    <tr>书籍数量</tr>
                    <tr>书籍详情</tr>
                    <tr>操作</tr>
                </thead>
                <tbody>
                    <c:forEach items="${bookList}" var="book">
                        <tr>
                            <td>${book.bookId}</td>
                            <td>${book.bookName}</td>
                            <td>${book.bookCounts}</td>
                            <td>${book.detail}</td>
                            <td>
                                <a href="#">修改</a> | &nbsp;
                                <a href="#">删除</a>
                            </td>
                        </tr>
                    </c:forEach>
                </tbody>
            </table>
        </div>
    </div>

</body>
</html>
```

4.写addBook，添加书籍到数据库  

```java
/**
 * addBook添加书籍
 * @param model
 * @return
 */
@PostMapping("/book/addBook")
public String addBook(Books books,Model model) {
    System.out.println(books);
    bookService.addBook(books);

    return "redirect:/book/allBook";
}
```

### 2.10.4、修改书籍

1.allBook.jsp增加修改样式和按钮

```java
<table class="table table-hover table-striped">
  <thead>
      <tr>书籍编号</tr>
      <tr>书籍名称</tr>
      <tr>书籍数量</tr>
      <tr>书籍详情</tr>
      <tr>操作</tr>
  </thead>
  <tbody>
      <c:forEach items="${bookList}" var="book">
          <tr>
              <td>${book.bookId}</td>
              <td>${book.bookName}</td>
              <td>${book.bookCounts}</td>
              <td>${book.detail}</td>
              <td>
                  <a href="#">修改</a> | &nbsp;
                  <a href="#">删除</a>
              </td>
          </tr>
      </c:forEach>
  </tbody>
</table>
```

2.增加跳转修改界面和修改功能

```java
/**
 * 跳转updateBook.jsp
 * @return
 */
@GetMapping("/book/toUpdateBookPiper")
public String toUpdateBookPiper() {
    return "updateBook";
}

/**
 * updateBook修改书籍
 *
 * @param model
 * @return
 */
@PostMapping("/book/updateBook")
public String updateBook(Books books, Model model) {
    System.out.println("/book/updateBook");
    System.out.println(books);
    bookService.updateBook(books);

    return "redirect:/book/allBook";
}
```

3.修改界面updateBook.jsp

```java
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/3
  Time: 16:04
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>修改书籍</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

    <div class="container">
        <div class="row clearfix">
            <div class="col-md-12 column">
                <div class="page-header">
                    <h1>
                        <small>修改书籍</small>
                    </h1>
                </div>
            </div>
        </div>

        <form action="/book/updateBook" method="post">
            <input type="text" class="form-control hidden" name="bookId" value="${book.bookId}"/>
            <div class="form-group">
                <label>书籍名称：</label>
                <input type="text" class="form-control" name="bookName" value="${book.bookName}" required/>
            </div>
            <div class="form-group">
                <label>书籍数量：</label>
                <input type="text" class="form-control" name="bookCounts" value="${book.bookCounts}" required/>
            </div>
            <div class="form-group">
                <label>书籍描述：</label>
                <input type="text" class="form-control" name="detail" value="${book.detail}" required/>
            </div>
            <div class="form-group">
                <input type="submit" class="form-control" value="修改"/>
            </div>
        </form>
    </div>

</body>
</html>
```

### 2.10.4、删除书籍

1.在BookController新增deleteBook

```java
/**
 * deleteBook删除书籍
 *
 * @param model
 * @return
 */
@GetMapping("/book/deleteBook/{bookId}")
public String deleteBook(@PathVariable("bookId") int id, Model model) {
    bookService.deleteBookById(id);
    return "redirect:/book/allBook";
}
```

2.allBook.jsp

```java
<a href="/book/deleteBook/${book.bookId}" class="btn btn-danger">删除</a>
```

### 2.10.5、搜索书籍

1.allBook.jsp

```java
<div class="row clearfix">
    <div class="col-md-2 column">
        <a href="/book/toAddBookPiper" class="btn btn-success">添加数据</a>
    </div>
    <div class="col-md-5 column">
        <form action="/book/searchBook" method="post" class="form-inline" role="form">
            <div class="form-group">
                <label>书籍名称：</label>
                <input type="text" class="form-control" name="bookName" placeholder="书籍名称" required/>
            </div>
            <input type="submit" value="查询" class="btn">
            <a href="/book/allBook" class="btn btn-success">重置</a>
        </form>
    </div>
</div>
```

2.BookMapper.java

```java
/**
 * 查询一本书name
 * @param bookName
 * @return
 */
List<Books> queryBookByBookName(@Param("bookName")String bookName);
```

3.BookMapper.xml

```java
<select id="queryBookByBookName" resultType="books" parameterType="string">
    select bookId,bookName,bookCounts,detail
    from books
    where bookName like '%' #{bookName} '%'
</select>
```

4.BookService.java

```java
/**
 * 查询一本书name
 *
 * @param bookName
 * @return
 */
List<Books> queryBookByBookName(String bookName);
```

5.BookServiceImpl.java

```java
@Override
public List<Books> queryBookByBookName(String bookName) {
    return bookMapper.queryBookByBookName(bookName);
}
```

6.BookController.java

```java
/**
 * 查询书籍根据书名
 *
 * @param model
 * @return
 */
@PostMapping("/book/searchBook")
public String searchBook(String bookName, Model model) {
    System.out.println(bookName);
    List<Books> bookSearchList = bookService.queryBookByBookName(bookName);
    for (Books books : bookSearchList) {
        System.out.println(books.toString());
    }
    model.addAttribute("bookList", bookSearchList);

    return "allBook";
}
```

## 2.11、拦截器

拦截器是AOP思想的应用，是横切进去的。

拦截器是SpirngMVC自己的

只会拦截控制器的方法，如果访问js/html/css/image/jsp/是不会拦截的，就是自带静态资源过滤。

### 2.11.1、自定义拦截器

自定义拦截器，需要实现HandlerInterceptor接口。

1、**springmvc-06-interceptor**,添加web支持

2、配置web.xml、springmvc.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--1.DispatcherServlet-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--2.乱码过滤-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--3.默认Session过期时间 15分钟-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
</web-app>

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <!--1.注解驱动-->
    <mvc:annotation-driven />

    <!--2.静态资源过滤-->
    <mvc:default-servlet-handler />

    <!--3.扫描包controller-->
    <context:component-scan base-package="com.kuang.controller"/>

    <!--4.视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--前缀:WEB-INF下的jsp包-->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

3、新建MyInterceptor.java

```java
package com.kuang.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 郭宇航
 * @date 2021/10/5
 * @apiNote
 */
public class MyInterceptor implements HandlerInterceptor {

    /**
     * 前置
     * 返回true就是放行，执行下一个拦截器
     * 返回false就是拦截，不执行下一个拦截器
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("=====处理前=======");
        return true;
    }

    /**
     * 核心
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("=====处理后======");
    }

    /**
     * 后置
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("=====清理=====");
    }
}
```

4、配置spring-mvc.xml

```java
<mvc:interceptors>
    <mvc:interceptor>
        <!--/**代表/**/admin/ 只要在**后面的都会拦截，直接写/a不会拦截/a/下面的/a/b-->
        <mvc:mapping path="/**"/>
        <bean class="com.kuang.interceptor.MyInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

5、测试[http://localhost:8080/test](http://localhost:8080/test)结果

```java
=====处理前=======
test
=====处理后======
=====清理=====
```

6、修改MyInterceptor.java

```java
package com.kuang.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 郭宇航
 * @date 2021/10/5
 * @apiNote
 */
public class MyInterceptor implements HandlerInterceptor {

    /**
     * 前置
     * 返回true就是放行，执行下一个拦截器
     * 返回false就是拦截，不执行下一个拦截器
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("=====处理前=======");
        return false;
    }

    /**
     * 拦截日志
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("=====处理后======");
    }

    /**
     * 后置
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("=====清理=====");
    }
}
```

7、测试[http://localhost:8080/test](http://localhost:8080/test)结果

```java
=====处理前=======
```

直接在拦截前preHandle方法中拦截死了。

### 2.11.2、自定义拦截器-例子-登录验证

实现非登录禁止进入main.jsp

1、新建WEB-INF/jsp/main.jsp

```java
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/5
  Time: 22:13
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>用户中心</title>
</head>
<body>

欢迎${userLoginInfo}

</body>
</html>
```

2、新建WEB-INF/jsp/login.jsp

```java
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/5
  Time: 22:12
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录页面</title>
</head>
<body>

<form action="/login" method="post">
    用户名：
    <input type="text" name="username">
    密码：
    <input type="text" name="password">
    <input type="submit" value="登录">
</form>

</body>
</html>
```

3、index.jsp

```java
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/5
  Time: 21:51
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>主页</title>
  </head>
  <body>
  <a href="/goLogin">登录</a>
  <a href="/main">用户中心</a>
  <a href="/signOut">退出</a>
  </body>
</html>
```

4、新建LoginController

```java
package com.kuang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

/**
 * @author 郭宇航
 * @date 2021/10/5
 * @apiNote
 */
@Controller
public class LoginController {

    @PostMapping("/login")
    public String login(HttpSession session, String username, String password) {
        session.setAttribute("userLoginInfo",username);
        //登录成功
        return "main";
    }

    @GetMapping("/signOut")
    public String signOut(HttpSession session) {
        session.removeAttribute("userLoginInfo");
        return "login";
    }

    @GetMapping("/main")
    public String main() {
        return "main";
    }

    @GetMapping("/goLogin")
    public String goLogin() {
        return "login";
    }
}
```

5、LoginInterceptor

```java
package com.kuang.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 郭宇航
 * @date 2021/10/5
 * @apiNote
 */
public class LoginInterceptor implements HandlerInterceptor {

    /**
     * 前置:判断什么情况下没有登录
     * 用户没有登录应该重定向，而不是false
     * 判断session
     * 返回true就是放行，执行下一个拦截器
     * 返回false就是拦截，不执行下一个拦截器
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("=====处理是否登录=======");
        //已登录时放行
        if (request.getSession().getAttribute("userLoginInfo") != null) {
            return true;
        }
        //前往登录界面时放行
        if (request.getRequestURI().contains("ogin")){
            return true;
        }
        //否则重定向到登录
        response.sendRedirect("/goLogin");
        return false;
    }
}
```

6、配置拦截器spring-mvc.xml

```java
<mvc:interceptors>
    <mvc:interceptor>
        <!--/**代表/**/admin/ 只要在**后面的都会拦截，直接写/a不会拦截/a/下面的/a/b-->
        <mvc:mapping path="/**"/>
        <bean class="com.kuang.interceptor.LoginInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

## 2.12、文件上传下载

**springmvc-07-file**

- springmvc可以很好的支持文件上传，但是SpringMVC上下文中默认没有装配MultipartResolver，因此默认情况下其不能处理文件上传工作。如果想使用Spring的文件上传功能，**需要在上下文中配置MultipartResolver。**
- 2003年，Apache Software Foundation发布开源的Commons FileUpload组件。
- Servlet3.0规范已经提供方法来处理文件上传，但这种上传需要在Servlet中完成。
- 而SpringMVC则提供简单的封装。
- SpringMVC为文件上传提供了直接的支持，这种支持是用即插即用的MultipartResolver实现的。
- **SpringMVC使用Commons FileUpload组件技术实现了一个MultipartResolver实现类：**
    - **CommonsMultipartResolver**
- **CommonsMultipartResolver常用方法：**
    - **String getOriginalFilename()：获取上传文件的原名**
    - **InputStream getInputStream()：获取文件流**
    - **void transferTo(File dest)：将上传文件保存到一个文件目录中**

**前端表单要求：**为了支持上传，表单的method设置为POST，并将enctype设置为multpart/form-data。只有在这样情况下，浏览器才会把用户选择的文件比二进制数据发送给服务器。

**表单中enctype属性详细说明：**

multipart/form-data是指表单数据有多部分构成：既有文本数据，又有文件等二进制数据的意思。

默认情况下，enctype的值是application/x-www-form-urlencoded，不能用于文件上传；

只有使用了multipart/form-data，才能完整的传递文件数据。

对于文件上传工作，其实是在前端完成的

**依赖：**

```java
<!--文件上传-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
<!--servlet-api导入高版本的-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
```

### 2.12.1、文件上传下载案例

配置bean:MultipartResolver

bean的id必须为multipartResolver，否则上传会被400

1.spring-mvc.xml

```java
<!--文件上传配置-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!--请求的编码格式,必须和jsp的pageEncoding属性一致，一边正确读取表单的内容，默认ISO-8859-1-->
    <property name="defaultEncoding" value="utf-8"/>
    <!--上传文件的大小限制，单位字节10485760=10M-->
    <property name="maxUploadSize" value="10485760"/>
    <property name="maxInMemorySize" value="40960"/>
</bean>
```

2.index.jsp

```java
<%--
  Created by IntelliJ IDEA.
  User: 郭宇航
  Date: 2021/10/5
  Time: 22:45
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>

  <form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="上传">
  </form>

  </body>
</html>
```

3.FileController

```java
package com.kuang.controller;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

import javax.servlet.http.HttpServletRequest;
import java.io.*;

/**
 * @author 郭宇航
 * @date 2021/10/5
 * @apiNote
 */
@RestController
public class FileController {

    /**
     * 第一种方式
     * @RequestParam("file")将name=file空间得到的文件封装成CommonsMultipartFile对象
     * 批量上传CommonsMultipartFile变为数组即可
     * @param file
     * @param request
     * @return
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("file") CommonsMultipartFile file, HttpServletRequest request) throws IOException {
        //获取文件名
        String uploadName = file.getOriginalFilename();
        System.out.println("文件名："+uploadName);
        //判断是否为空
        if (uploadName == null) {
            return "redirect:/index.jsp";
        }

        //设置保存上传路径
        String path = request.getServletContext().getRealPath("/upload");
        //如果路径不存在创建
        File realPath = new File(path);
        if (!realPath.exists()) {
            realPath.mkdirs();
        }
        System.out.println("文件保存路径："+realPath);

        //上传文件的输入流
        InputStream inputStream = file.getInputStream();
        //创建文件输出流
        OutputStream outputStream = new FileOutputStream(new File(realPath,uploadName));

        //读取和写入
        int len = 0;
        byte[] buffer = new byte[1024];
        while ((len = inputStream.read(buffer)) != -1) {
            outputStream.write(buffer, 0, len);
            outputStream.flush();
        }
        outputStream.close();
        inputStream.close();

        return "redirect:/index.jsp";
    }

    @PostMapping("/upload2")
    public String fileUpload(@RequestParam("file") CommonsMultipartFile file, HttpServletRequest request) {
        //上传路径,获取运行时的工程存放目录
        String path = request.getServletContext().getRealPath("/upload");
        File realPath = new File(path);
        if (!realPath.exists()) {
            realPath.mkdirs();
        }
        System.out.println("文件保存路径："+realPath);

        //通过transferTo写入文件
        try {
            file.transferTo(new File(realPath+"/"+file.getOriginalFilename()));
        } catch (IOException e) {
            System.out.println("文件保存错误");
            e.printStackTrace();
        }

        return "redirect:/index.jsp";
    }
}
```

4.文件下载

```java
@GetMapping("/download")
public String download(HttpServletResponse response, HttpServletRequest request) throws IOException {
    //要下载的图片
    String path = request.getServletContext().getRealPath("/upload");
    String filename = "新建文本文档.txt";

    //1.设置response响应头
    response.reset();//设置页面不缓存，情况buffer
    response.setCharacterEncoding("UTF-8"); //字符编码
    response.setContentType("multipart/form-data"); //二进制传输数据
    response.setHeader("Content-Disposition","attachment;fileName="+ URLEncoder.encode(filename,"UTF-8"));

    //2.读取文件
    File file = new File(path,filename);
    InputStream inputStream = new FileInputStream(file);
    
    //3.写出文件
    OutputStream outputStream = new FileOutputStream(file);
    byte[] buffer = new byte[1024];
    int index = 0;
    while ((index = inputStream.read(buffer)) != -1) {
        outputStream.write(buffer, 0, index);
        outputStream.flush();
    }
    outputStream.close();
    inputStream.close();
    return "ok";
}
```