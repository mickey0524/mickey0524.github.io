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
	
* 在 Java 配置（JavaConfig）中，可以使用 @Profile 注解指定某个 bean 属于哪一个 profile，@Profile 可以用于类，也可以用于方法，例如，`@Profile("dev")` 和 `@Profile("prod")`

* 在 XML 配置中，可以定义带有 profile 选项的 beans，然后根据 `spring.profiles.active` 和 `spring.profiles.default` 两个属性来选择使用的配置

* Spring 4 之后，引入了一个 @Conditional 注解，只有在给定的条件为 true 的时候，才会创建这个 bean，否则的话，这个 bean 会被忽略
	
	传入 @Conditional 注解的类需要实现 Condition 接口
	
	```java
	public interface Condition {
		boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
	}
	
	public class MagicExistsCondition implements Condition {
		public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
			Environment env = context.getEnvironment();
			return env.containsProperty("magic");  // 只有环境变量中存在 magic 属性的时候，才返回 true，bean 才被加载
		}
	}
	```
	
* Spring 支持使用 @Autowired 来自动装配，如果这个时候有多个 bean 符合装配的条件，Spring 会抛出异常，这个时候我们需要显示的告诉 Spring 我们想要哪个 bean

	* 使用 @Primary 或在 XML 中设置 primary 来表明优先选择这个 bean

		```java
		@Component
		@Primary
		public class IceCream implements Dessert { ... }
		
		<bean id="iceCream" class="com.IceCream" primary="true">
		```
		
	* 同时在 @Component 和 @Autowired 处使用 @Qualifier

		```java
		@Component
		@Qualifier("iceCream")
		public class IceCream implements Dessert { ... }
		
		@Autowired
		@Qualifier("iceCream")
		public void setDessert(Dessert dessert) {
			this.dessert = dessert;
		}
		```
	
	* 和前一个差不多，自定义注解继承 @ Qualifier，可以组合着使用

		```java
		@Target({ElementType.CONSTRUCTOR, ElementType.FIELD,
					ElementType.METHOD, ElementType.TYPE})
		@Retention(RetentionPolicy.RUNTIME)
		@Qualifier
		public @interface A { }
		
		@Target({ElementType.CONSTRUCTOR, ElementType.FIELD,
					ElementType.METHOD, ElementType.TYPE})
		@Retention(RetentionPolicy.RUNTIME)
		@Qualifier
		public @interface B { }
		
		@Target({ElementType.CONSTRUCTOR, ElementType.FIELD,
					ElementType.METHOD, ElementType.TYPE})
		@Retention(RetentionPolicy.RUNTIME)
		@Qualifier
		public @interface C { }
		
		@Component
		@A
		@B
		public class IceCream implements Dessert { ... }
		
		@Autowired
		@A
		@B
		public void setDessert(Dessert dessert) {
			this.dessert = dessert;
		}
		```

* Spring 中 bean 的作用域默认是单例，我们前面也讲到过，但是有时候我们可以选择其他作用域，可以使用 @Scope 来修改 bean 的作用域，或者在 XML 中设置 scope

	* 单例（Singleton）：在整个应用中，只创建 bean 的一个实例
	* 原型（Prototype）：每次注入或者通过 Spring 应用上下文获取的时候，都会创建一个新的 bean 实例

		```java
		@Component
		@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
		public class Notepad { ... }
		
		<bean id="notepad" class="com.Notepad" scope="prototype" />
		```
		
	* 会话（Session）：在 Web 应用中，为每个会话创建一个 bean 实例
		
		* 基于接口 —— 走动态代理

			```java
			@Scope(value=WebApplicationContext.SCOPE_SESSION,
					 proxyMode=ScopedProxyMode.INTERFACES)
					 
			<bean id="cart"
					class="com.Cart"
					scope="session">
				<aop:scoped-proxy proxy-target-class="false" />
			</bean>
			```

		* 基于类 —— 走 CGLib
			
			```java
			@Scope(value=WebApplicationContext.SCOPE_SESSION,
					 proxyMode=ScopedProxyMode.TARGET_CLASS)

			<bean id="cart"
					class="com.Cart"
					scope="session">
				<aop:scoped-proxy />
			</bean>
			```
			
	* 请求（Request）：在 Web 应用中，为每个请求创建一个 bean 实例

		
	