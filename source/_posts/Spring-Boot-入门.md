---
title: Spring Boot 入门
date: 2017-07-26 00:15:49
categories: coding
tags:
  - Java
  - Spring Boot
---

这篇文章是我学习 Spring Boot 的总结

<!--more-->

## Spring Boot介绍

Spring Boot简化了基于Spring的应用开发，你只需要"run"就能创建一个独立的，产品级别的Spring应用。 

提供一系列大型项目常用的非功能性特征，比如：内嵌服务器，安全，指标，健康检测，外部化配置。

### 为什么使用 JavaBean

Java语言欠缺属性、事件、多重继承功能。所以，如果要在Java程序中实现一些面向对象编程的常见需求，只能手写大量胶水代码。Java Bean正是编写这套胶水代码的惯用模式或约定。

这些约定包括getXxx、setXxx、isXxx、addXxxListener、XxxEvent等。

[Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)

### 基于JavaBeans的采用控制反转

控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入(DI)，还有一种方式叫“依赖查找”。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

所谓的依赖注入，则是，甲方开放接口，在它需要的时候，能够讲乙方传递进来(注入)
所谓的控制反转，甲乙双方不相互依赖，交易活动的进行不依赖于甲乙任何一方，整个活动的进行由第三方负责管理。

Class A中用到了Class B的对象b，一般情况下，需要在A的代码中显式的new一个B的对象。

采用依赖注入技术之后，A的代码只需要定义一个私有的B对象，不需要直接new来获得这个对象，而是通过相关的容器控制程序来将B对象在外部new出来并注入到A类里的引用中。而具体获取的方法、对象被获取时的状态由配置文件（如XML）来指定。


IoC 的思想最核心的地方在于，资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很多好处。第一，资源集中管理，实现资源的可配置和易管理。第二，降低了使用资源双方的依赖程度，也就是我们说的耦合度。

[Spring IoC有什么好处呢？](https://www.zhihu.com/question/23277575)

#### Spring Beans和依赖注入

你可以自由地使用任何标准的Spring框架技术去定义beans和它们注入的依赖。简单起见，我们经常使用 `@ComponentScan` 注解 **搜索 beans**，并结合 `@Autowired` 构造器注入。

如果遵循以上的建议组织代码结构（将应用的main类放到包的最上层，即root package），那么你就可以添加 `@ComponentScan` 注解而不需要任何参数，所有应用组件（@Component, @Service, @Repository, @Controller等）都会自动注册成Spring Beans。



### Spring Boot 安装

对于java开发者来说，使用Spring Boot就跟使用其他Java库一样，只需要在你的classpath下引入适当的 `spring-boot-*.jar` 文件。

Spring Boot依赖使用的 groupId 为 org.springframework.boot。通常，你的Maven POM文件会继承 `spring-boot-starter-parent` 工程，并声明一个或多个“Starter POMs”依赖。

### 编写代码

Maven默认会编译 `src/main/java` 下的源码

```
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```

#### @RestController和@RequestMapping注解

`@RequestMapping` 注解提供路由信息，它告诉Spring任何来自 "/" 路径的HTTP请求都应该被映射到 `home` 方法。

`@RestController` 注解告诉 Spring 以字符串的形式渲染结果，并直接返回给调用者。

#### @EnableAutoConfiguration注解

第二个类级别的注解是 `@EnableAutoConfiguration`，这个注解告诉Spring Boot根据添加的jar依赖猜测你想如何配置Spring。由于 `spring-boot-starter-web` 添加了Tomcat和Spring MVC，所以auto-configuration将假定你正在开发一个web应用，并对Spring进行相应地设置。

#### main方法

应用程序的最后部分是 `main方法，这是一个标准的方法，它遵循Java对于一个应用程序入口点的约定。我们的 `main` 方法通过调用 `run`，将业务委托给了Spring Boot的 `SpringApplication` 类。`SpringApplication` 将引导我们的应用，启动Spring，相应地启动被自动配置的Tomcat web服务器。我们需要将Example.class作为参数传递给run方法，以此告诉 `SpringApplication` 谁是主要的Spring组件，并传递args数组以暴露所有的命令行参数。

#### 使用@SpringBootApplication注解

很多Spring Boot开发者经常使用 `@Configuration`，`@EnableAutoConfiguration`，`@ComponentScan` 注解他们的main类

`@SpringBootApplication` 注解等价于以默认属性使用 `@Configuration`，`@EnableAutoConfiguration` 和 `@ComponentScan`

## Spring MVC和 Spring

Spring可以说是一个管理bean的容器，也可以说是包括很多开源项目的总称，spring mvc是其中一个开源项目，所以简单走个流程的话，http请求一到，由容器（如：tomact）解析http搞成一个request，通过映射关系（路径，方法，参数啊）被spring mvc一个分发器去找到可以处理这个请求的bean，那tomcat里面就由spring管理bean的一个池子（bean容器）里面找到，处理完了就把响应返回回去。

Spring最早就一个项目，它是基于反射机制实现的，目的就是提供IOC依赖注入容器，通过它可以获得类对象。

为了实现IOC，最基础的就是在xml种配置一个一个的bean，IOC将根据xml中的配置（主要是类名）返回对应的对象。

一般来说，这个对象内有一些属性需要初始化，那么传统做法就是在xml对应bean下继续配置属性，这样IOC会继续分析每个属性对应哪个bean，再根据xml分配这些依赖的bean。

但是后来出了注解，那么就可以取代在xml中配置属性的这个做法了（配置类还是要的），可以直接在对应的class中为每个属性写一个注解，即标明它应该赋值为哪个bean，这样IOC在运行时创建对象会首先反射分析这个类的每个属性上的注解，从而得知应该用哪个bean来赋值，这就是spring注解的用途了。

* [详解 Spring 3.0 基于 Annotation 的依赖注入实现](https://www.ibm.com/developerworks/cn/opensource/os-cn-spring-iocannt/)
* [ Spring的xml配置bean文件原理-[Java反射机制]](http://blog.csdn.net/zhang6622056/article/details/7659489)

## Spring Boot和 Spring MVC

Spring MVC是Spring核心框架的一个模块，它是一个Web框架，和它类似像Struts。它解决的问题领域是网站应用程序或者服务开发——URL路由、Session、模板引擎、静态Web资源等等。

Spring Boot是这几年微服务概念流行后，Spring开发的一套快速开发Spring应用的框架。

它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具。同时它集成了大量常用的第三方库配置（例如Jackson, JDBC, Mongo, Redis, Mail等等），Spring Boot应用中这些第三方库几乎可以零配置的开箱即用（out-of-the-box）

## Spring boot与 Spring cloud 

Spring Boot就是一个内嵌Web服务器（tomcat/jetty）的可执行程序的框架。你开发的web应用不需要作为war包部署到web服务器中，而是作为一个可执行程序，启动时把Web服务器配置好，加载起来。

Spring Boot比较适合微服务部署方式，不再是把一堆应用放到一个Web服务器下，重启Web服务器会影响到其他应用，而是每个应用独立使用一个Web服务器，重启动和更新都很容易。比如 REST 和 RabbitMQ 组件

Spring Cloud是一套微服务开发和治理框架，来自Netflex的OSS，包含了微服务运行的功能，比如远程过程调用，动态服务发现，负载均衡，限流等。





## 参考

* [Spring-Boot-Reference-Guide](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/content/)
* [Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)
* [Spring IoC有什么好处呢？](https://www.zhihu.com/question/23277575)
* [SpringMVC和Spring是什么关系？](https://www.zhihu.com/question/39678061)





