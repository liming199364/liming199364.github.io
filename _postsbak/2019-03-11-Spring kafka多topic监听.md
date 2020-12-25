---
layout: post 
title: 'Spring kafka多topic监听'
date: 2019-03-11
categories: Spring
tags: Spring
---

今天遇到一个问题，还是自己对Spring不太熟悉

```java
@KafkaListener(topics = "hello")
```

以前总是监听一个topic，现在发现监听多个topic的时候，无从着手。

```java
	/**
	 * The topics for this listener.
	 * The entries can be 'topic name', 'property-placeholder keys' or 'expressions'.
	 * Expression must be resolved to the topic name.
	 * Mutually exclusive with {@link #topicPattern()} and {@link #topicPartitions()}.
	 * @return the topic names or expressions (SpEL) to listen to.
	 */
	String[] topics() default {};
```

topics属性是一个数组，那第一时间想到的就是

```java
@KafkaListener(topics = {"hello","hello2","hello3"})
```

转换为动态配置就是

```java
@KafkaListener(topics = {"${kafka.hello1}","${kafka.hello2}","${kafka.hello3}"})
```

还是好蠢，和手动填入没什么区别

注释里面有一句or expressions (SpEL) to listen to.

于是查找了下Spring的SpEL语法，直接告诉解决方案吧

```java
@KafkaListener(topics = "#{'${kafka.orginTopic}'.split(',')}")
orginTopic: "world,hello"
```

topics解析源码如下

```java
	private Object resolveExpression(String value) {
		String resolvedValue = resolve(value);

		if (!(resolvedValue.startsWith("#{") && value.endsWith("}"))) {
			return resolvedValue;
		}

		return this.resolver.evaluate(resolvedValue, this.expressionContext);
	}    
```

什么是SpEL语法

### SpEL语法

Spring表达式语言全称为Spring Expression Language(缩写为SpEL)，能在运行时构建复杂表达式、存取对象图属性、对象方法调用等等，并且能与Spring功能完美整合，如能用来配置Bean定义。 表达式语言给静态Java语言增加了动态功能。

SpEL是单独模块，只依赖于core模块，不依赖于其他模块，可以单独使用。

**在Bean配置定义中，可以直接通过"#{}"编写SpEL表达式。**



具体包含多少强大的功能，可以看下参考~


### 参考

[Spring SpEL语法](http://xiaobaoqiu.github.io/blog/2015/04/09/spring-spelyu-fa/)

[说说 Spring 表达式语言（SpEL）中的各种表达式类型](https://juejin.im/post/5b9cb6825188255c5546e5f4#heading-8)