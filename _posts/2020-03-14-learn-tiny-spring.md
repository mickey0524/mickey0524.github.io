---
layout:     post
title:      "learn-tiny-spring"
subtitle:   "学习微型 Spring"
date:       2020-03-14 15:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - java
---

Spring 的源码非常复杂，github 上的 [tiny-spring](https://github.com/code4craft/tiny-spring/) 项目简要实现了 Spring 的功能，这里做一些笔记，另外[tiny-spring分析](https://www.zybuluo.com/dugu9sword/note/382745)是别人做的分析，解释了项目中每个类的作用，可以进行参考

# step-1-container-register-and-get

step-1 是最基本的，仅仅实现了 bean 的注册和获取

我们知道 Spring 是面向 Bean 的，Bean 包裹了一个 Object 对象，在项目中 `BeanDefinition.java` 定义了 Bean，然后 `BeanFactory.java` 使用一个 Map 注册管理项目中的 Beans

# step-2-abstract-beanfactory-and-do-bean-initilizing-in-it

step-2 将 BeanFactory 定义为接口，然后抽象了 AbstractBeanFactory 这个抽象类，定义了 AutowireCapableBeanFactory 子类

扩展 BeanDefinition 类，新增 beanClass 和 beanClassName 属性，我们知道，在 XML 中定义 bean 的时候需要提供 class

通过 `Class.forName` 获取类，然后通过 `beanClass.newInstance` 获取实例

# step-3-inject-bean-with-property

step-3 在 step-2 的基础上，增加了对 Bean 中 property 的处理，实现了将 property 写入 Bean 中 Object 对象的功能

在 `beanClass.newInstance` 获取实例之后，通过 `class.getDeclaredField` 获取 Field 实例，再通过 `field.set` 方法写入

# step-4-config-beanfactory-with-xml

step-4 主要增加了如何读取 xml 文件，其实就是 xpath 解析，首先将 XML 文件读成 InputStream，然后使用 Document 解析 InputStream，最后依次解析 `<bean>` 和 `<property>`

# step-5-inject-bean-to-bean

step-5 主要增加了 XML 中 ref 依赖其他 bean 的方法，其实就是在解析 `<property>` 时候，当 value 不存在的时候，新增 ref 的判断，然后在 BeanFactory 实例化 Bean 对象的时候，从 BeanFactory 的 beanMap 中取出 ref 依赖的 Bean 对象

# step-6-invite-application-context

step-6 在之前的基础上新增了 ApplicationContext，用于管理应用的上下文，ApplicationContext 首先读取 XML 文件，将 Bean 读取至 BeanDefinitionReader 中的 Map 中，然后将 Map copy 至 BeanFactory 的 Map 中

# step-7-method-interceptor-by-jdk-dynamic-proxy

step-7 增加了 JDK 动态代理类型的 AOP Proxy，TargetSource 中存储了需要被代理的对象，AdvisedSupport 中存储了 TargetSource 以及 MethodInceptor，MethodInceptor 是一个接口，用于在被代理对象方法执行前后自定义逻辑，MethodInceptor 接受 MethodInvocation 作为参数，MethodInvocation 内的逻辑就是保证被代理对象方法的正常执行

至于如何进行动态代理，自然是实现 InvocationHandler 接口，使用 Proxy.newProxyInstance 啦

# step-8-invite-pointcut-and-aspectj

step-8 增加了 PointCut 相关的东西，使得 AOP 能够借助 AspectJ 实现 execution，进而判断目标 class 和目标 method 是否 match 切点

# step-9-auto-create-aop-proxy

step-9 增加了切点对 Bean 实例和 Bean 实例中方法的 match

ApplicationContext 首先调用 XmlBeanDefinitionReader 读取 XML 文件，将所有的 Bean 定义读取为 BeanDefinition 放置于 AbstractBeanDefinitionReader 中的 registry 这个 Map 中，然后将 kv 拷贝到 BeanFactory 中；然后注册 BeanPostProcessors，BeanPostProcessors 是用于在 Bean 实例化的时候进行切点的判断，对 match 的 Bean 实例进行 JDK Proxy

# step-10-invite-cglib-and-aopproxy-factory

step-10 新增 cglib 方式和 AOP 代理工厂

