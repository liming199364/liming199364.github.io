---
layout: post 
title: 'Spring Boot集成proguard混淆'
date: 2018-11-13
categories: Spring
tags: Java
---

> 众所周知，Java工程很容易反编译，那么保证代码不被很容易解析的方法就是proguard

最近项目需要在spring boot工程中对代码进行混淆，找了很多资料，最终找到一个适合自己的方案。

**核心思想**：guard会将函数名，成员等替换为a,b,c等进行混淆，当工程中存在反射的情形（比如依赖注入，注解等等，需要keep住，保证混淆的时候不被a,b,c等替换）



#### pom.xml加入plugin

```xml
<plugin>
    <groupId>com.github.wvengen</groupId>
    <artifactId>proguard-maven-plugin</artifactId>
    <version>2.0.14</version>
    <executions>
        <execution>
            <id>proguard</id>
            <phase>package</phase>
            <goals>
                <goal>proguard</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <proguardInclude>${basedir}/../proguard.cfg</proguardInclude>
        <libs>
            <lib>${java.home}/lib/rt.jar</lib>
        </libs>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>net.sf.proguard</groupId>
            <artifactId>proguard-base</artifactId>
            <version>5.3.3</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
</plugin>
```

这里面引用到了父工程的一个配置文件proguard.cfg

#### 外部加入proguard.cfg

```
-target 1.8 ##指定java版本号
-dontshrink ##默认是开启的，这里关闭shrink，即不删除没有使用的类/成员
-dontoptimize ##默认是开启的，这里关闭字节码级别的优化
-useuniqueclassmembernames ##对于类成员的命名的混淆采取唯一策略
-adaptclassstrings ## 混淆类名之后，对使用Class.forName('className')之类的地方进行相应替代
-dontusemixedcaseclassnames ## 混淆时不生成大小写混合的类名，默认是可以大小写混合

-keepattributes Exceptions,InnerClasses,Signature,Deprecated,SourceFile,LineNumberTable,*Annotation*,EnclosingMethod ##对异常、注解信息在runtime予以保留，不然影响springboot启动
-keepclasseswithmembers public class * { public static void main(java.lang.String[]);} ##保留main方法的类及其方法名
-keepclassmembers enum * { *; }  ##保留枚举成员及方法

-keepclassmembers class * {
                                @org.springframework.beans.factory.annotation.Autowired *;
                                @org.springframework.beans.factory.annotation.Value *;
                                @org.apache.ibatis.annotations.Mapper *;
                                @org.springframework.scheduling.annotation.Scheduled *;
                                @org.springframework.stereotype.Component *;
                                @org.springframework.context.annotation.Configuration *;
                            }
#保留getter setter
-keepclassmembernames class * {
            void set*(***);
            void set*(int, ***);
            boolean is*();
            boolean is*(int);
            *** get*();
            *** get*(int);
        }
#保留serializable
-keepnames class * implements java.io.Serializable
-keepclassmembernames class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# 不警告
-dontwarn

# 不理会警告，否则混淆失败
-ignorewarnings

# 这里加入你不想被混淆的class
-keep class com.* {
    *;
}
```

#### bean名异常

不同包名下会有同样名称的a.class，这里需要加入

```java
@SpringBootApplication
public class MvcDemoApplication {

    public static class CustomGenerator implements BeanNameGenerator {

        @Override
        public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
            return definition.getBeanClassName();
        }
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(MvcDemoApplication.class)
                .beanNameGenerator(new CustomGenerator())
                .run(args);
    }
}
```

这里我没验证，大家可以试一试