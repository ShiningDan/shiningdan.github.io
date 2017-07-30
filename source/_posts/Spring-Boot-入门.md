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

控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入，还有一种方式叫“依赖查找”。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

Class A中用到了Class B的对象b，一般情况下，需要在A的代码中显式的new一个B的对象。

采用依赖注入技术之后，A的代码只需要定义一个私有的B对象，不需要直接new来获得这个对象，而是通过相关的容器控制程序来将B对象在外部new出来并注入到A类里的引用中。而具体获取的方法、对象被获取时的状态由配置文件（如XML）来指定。

[Spring IoC有什么好处呢？](https://www.zhihu.com/question/23277575)


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




## 参考

* [Spring-Boot-Reference-Guide](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/content/)
* [Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)
* [Spring IoC有什么好处呢？](https://www.zhihu.com/question/23277575)





