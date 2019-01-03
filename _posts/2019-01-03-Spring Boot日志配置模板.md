---
layout: post 
title: '消息中间件'
date: 2019-01-03
categories: Spring
tags: Spring
---
### logback-spring模板
留住备用
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration  scan="true" scanPeriod="60 seconds" debug="false">

    <contextName>res-config-operation</contextName>

    <property name="appid" value="15" scope="context"/>
    <property name="local" value="0" scope="context"/>
    <property name="cmdId" value="0" scope="context"/>
    <property name="subCmdId" value="0" scope="context"/>

    <property name="log.path" value="/data/data" />
    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--<pattern>%-30(%d{YYYY-MM-dd HH:mm:ss.SSS} [%contextName]) [%-5level][%logger{0}][%M][%L] : %msg%n</pattern>-->
            <pattern>%d{HH:mm:ss.SSS} [%level]%M.java\(%L\)|%logger{0}|%msg%n</pattern>
        </encoder>
    </appender>

    <!--输出到文件-->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/root_log.%d{yyyy-MM-dd}</fileNamePattern>
        </rollingPolicy>
        <encoder>
            <!--<pattern>%-30(%d{YYYY-MM-dd HH:mm:ss.SSS} [%contextName]) [%-5level][%logger{0}][%M][%L] : %msg%n</pattern>-->
            <pattern>%d{HH:mm:ss.SSS} [%level]%M.java\(%L\)|%logger{0}|%msg%n</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="console" />
        <appender-ref ref="file" />
    </root>

</configuration>
```