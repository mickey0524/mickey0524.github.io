---
layout:     post
title:      "begin-spring"
subtitle:   "Spring 小白入门"
date:       2019-11-16 15:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - java
---

这篇博客从零开始学习《Spring 实战》一书，做些笔记，用以温故知新

* Spring 依赖 @Component、@ComponentScan、@Autowired 实现自动装载 Bean，@Component 注解的类默认的 Bean id 为 类名的全小写，可以通过传值进行修改，@ComponentScan 默认扫描同包下的类，可以通过传值进行修改，可以传入字符串，代表包名，也可以传入类或接口，Spring 会去扫描这些类或接口所在的包，这也就可以单独定义一个空的 Scan 接口，代码风格会比较好

	```java
	package com;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	
	@Configuration
	@ComponentScan("com")
	// @ComponentScan(basePackages={"com", "com1"})
	// @ComponentScan(basePackageClasses={Mark.class})
	public class BeanConfig {
	}

	package com;
	
	import org.springframework.stereotype.Component;
	
	@Component
	public class Homework {
	
	    private String content;
	
	    public Homework() {
	        this.content = "数学";
	    }
	
	    public Homework(String content) {
	        this.content = content;
	    }
	
	    public String getContent() {
	        return content;
	    }
	
	    public void setContent(String content) {
	        this.content = content;
	    }
	}
	
	package com;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Component;
	
	@Component("customizedBeanId")  // 如果没有参数，Spring 加载的时候默认的 id 为 student
	public class Student {
	
	    private String name;
	
	    @Autowired
	    private Homework homework;
	
	    public Student() {
	        this.name = "sgy";
	    }
	
	    public void doHomeWork() {
	        System.out.println(homework.getContent());
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public Homework getHomework() {
	        return homework;
	    }
	
	    public void setHomework(Homework homework) {
	        this.homework = homework;
	    }
	}
	
	package com;
	
	import org.springframework.context.annotation.AnnotationConfigApplicationContext;
	
	public class App {
	
	    public static void main(String[] args) {
	        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanConfig.class);
	
	        Student student = context.getBean(Student.class);
	        System.out.println(student.getName() + "准备做作业了");
	        student.doHomeWork();
	        context.close();
	    }
	}
	```

* @Autowired 可以用在 Field 字段上，也可以使用在任意方法上（构造方法、set 方法都可以），如果没有匹配的 bean，Spring 会抛出异常，可以使用 `@ Autowired(required=false)`，这样会有 NPE 的风险，当有多个 bean 匹配的时候，Spring 会抛出异常，这个可以解决，见后面

* Spring 使用 JavaConfig 显示装配的时候，@Bean 注解的方法会被 Spring 捕获，返回唯一的 Bean 实例，例如下面的 homework 方法

	```java
	public class BeanConfig {
	
	    @Bean
	    Homework homework() {
	        return new Homework();
	    }
	
	    @Bean
	    Student student() {
	        return new Student(homework());
	    }
	}
	```
	
	更好的写法是
	
	```java
	public class BeanConfig {

	    @Bean
	    Homework homework() {
	        return new Homework();
	    }
	
	    @Bean
	    Student student(Homework homework) {
	        return new Student(homework);
	    }
	}
	```

* Spring 最原始的装配方式就是通过 XML，不能忘本呀，一个 `<bean>` 表情表示一个 bean，可以通过 `<constructor-arg>` 和 `<property>` 来设置属性，可以直接用 value 设置，也可以使用 ref 依赖其他 bean

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:aop="http://www.springframework.org/schema/aop"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans
	       http://www.springframework.org/schema/beans/spring-beans.xsd
	       http://www.springframework.org/schema/aop
	       http://www.springframework.org/schema/aop/spring-aop.xsd">
	    <!-- 让Spring管理 Student bean -->
	    <bean id="student" class="com.Student">
	        <property name="name" value="小明"/>
	        <property name="homework" ref="homework"/>
	    </bean>
	
	    <!-- 让Spring管理Homework bean-->
	    <bean id="homework" class="com.Homework">
	        <property name="content" value="how to calc 3 + 2"/>
	    </bean>
	
	    <!-- 切面定义-->
	    <bean id="checkTime" class="com.CheckNowTime"/>
	
	    <aop:config>
	        <aop:aspect ref="checkTime">
	            <aop:pointcut id="dohomework" expression="execution(* *.doHomeWork(..))"/>
	            <aop:before pointcut-ref="dohomework" method="beforDoHomework"/>
	            <aop:after pointcut-ref="dohomework" method="afterDoHomework"/>
	        </aop:aspect>
	    </aop:config>
	</beans>
	```

* Spring XML 装配引用其他装配

	* 引用其他 XML 装配

		```xml
		<import resource="A.xml">
		```
	
	* 引用一个 JavaConfig 类

		```xml
		<bean class="pathToConfig">
		```

* Spring JavaConfig 装配引用其他装配

	* 引用其他 JavaConfig 类

		```java
		@Configuration
		@Import(A.class)
		public class Config {
		}
		```
	
	* 引用其他 XML 装配

		```java
		@Configuration
		@ImportResource("classpath:config.xml")
		public class Config {
		}
		```