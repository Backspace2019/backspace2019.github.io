---
layout:     post
title:      "[JavaEE] Spring和SpringBoot"
subtitle:   "SpringBoot-2.2.2.RELEASE"
date:       2020-05-21 10:50:00
author:     "Backspace"
header-img: "img/post-bg-apple-event-2015.jpg"
catalog:      true
tags:
    - Spring
    - SpringBoot
---

### Spring是什么

Spring是一个**IOC**(DI)和**AOP**容器框架。控制反转，面向切面

**轻量级**：非侵入式	重量级：侵入式

可以管理对象生命周期的**容器**

### 三层架构

1. 控制层:`Controller`，完成客户端的交互，处理请求与响应(服务员)
2. 业务层:`Service`，完成业务逻辑的处理(厨师,厨房)
3. 持久层:`Dao`，完成与数据库的增删改查操作(MyBatis,采购,买菜)

### SpringBoot是什么

Spring是用来简化JAVAEE的SpringBoot是用来简化Spring的

### 使用IDEA开发SpringBoot程序

1. 在Idea中new→Module→Spring Initializr
2. 给工程命名、设置包名等，其他默认即可
3. 选择工程的版本
4. 搜索需要集成的模块
5. 点击Next ，给工程命名，然后点击Finish

### SpringBoot常用注解

- @SpringBootApplication    是Spring Boot核心注解，从这个注解启动springboot
- @Component    普通组件注解
- @Repository    持久层组件注解
- @Service    业务逻辑层注解
- @Controller    控制层注解
- @Autowired
- @Qualifier
- @Override
- @Test
- @ComponentScan
- @ResponseBody
- @RequestMapping("/getAllUser") //指定请求URL
- @RestController
- @MapperScan(basePackages = "com.atguigu.springboot.mapper")
- @PathVariable
- @WebFilter
- @ServletComponentScan

### Restful风格URL