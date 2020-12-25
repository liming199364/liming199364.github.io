---

layout: post 
title: 'Spring Boot 普通类使用Spring管理对象'
date: 2018-08-30 
categories: Spring 
tags: Java 

---
Spring Boot可以自动帮忙管理对象，通过Yml配置生成的Component全局唯一，但是@AutoWired必须配合@Controller或者@Service使用，如果一个普通new出来的类想要使用Spring Boot生成的对象，解决方案如下。

### 工具类

```java
/**
 * 非spring操作时需要使用spring容器已经加载的环境上下文
 */
@Component
public class ContextUtils implements ApplicationContextAware {
    
    private static ApplicationContext context;
    
    public static ApplicationContext getContext() {
        return context;
    }
    
    public static void setContext(ApplicationContext context) {
        ContextUtils.context = context;
    }
    
    /**
     * 获取spring已经加载的bean对象
     * 通过类class获取
     */
    public static <T> T getBeanByClass(Class<T> bean) throws Exception {
        return context.getBean(bean);
    }
    
    /**
     * 通过bean id 获取bean
     */
    public static Object getBeanById(String beanId) throws Exception {
        if (StringUtils.isNotBlank(beanId)) {
            return context.getBean(beanId);
        }
        return null;
    }
    
    /* (non-Javadoc)
     * @see org.springframework.context.ApplicationContextAware#setApplicationContext(org.springframework.context.ApplicationContext)
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    	System.out.println("---------------------------");
    	System.out.println("---------------------------");
    	
    	System.out.println("---------------------------");
    	System.out.println("---------------------------");
    	System.out.println("---------------------------");
    	ContextUtils.context = applicationContext;
    	System.out.println("========ApplicationContext配置成功,在普通类可以通过调用SpringUtils.getAppContext()获取applicationContext对象,applicationContext="+ContextUtils.context+"========");
    }
}
```

### 测试yml代码

```html
my:
  webserver:
    #HTTP 监听端口
    port: 80
    #嵌入Web服务器的线程池配置
    threadPool:
      maxThreads: 100
      minThreads: 8
      idleTimeout: 60000
```

### 测试类

```java
@Component
@ConfigurationProperties(prefix = "my.webserver")
public class ConsumerProperties {
    private int port;
    private ThreadPool threadPool;
    public int getPort() {
        return port;
    }
    public void setPort(int port) {
        this.port = port;
    }
    public ThreadPool getThreadPool() {
        return threadPool;
    }
    public void setThreadPool(ThreadPool threadPool) {
        this.threadPool = threadPool;
    }
    public static class ThreadPool {
        private int maxThreads;
        private int minThreads;
        private int idleTimeout;
        public int getIdleTimeout() {
            return idleTimeout;
        }
        public void setIdleTimeout(int idleTimeout) {
            this.idleTimeout = idleTimeout;
        }
        public int getMaxThreads() {
            return maxThreads;
        }
        public void setMaxThreads(int maxThreads) {
            this.maxThreads = maxThreads;
        }
        public int getMinThreads() {
            return minThreads;
        }
        public void setMinThreads(int minThreads) {
            this.minThreads = minThreads;
        }
    }
}
```

### 使用方式

	ConsumerProperties properties = ContextUtils.getBeanByClass(ConsumerProperties.class);

### 参考文章
[spring boot普通类使用spring管理的对象](https://blog.csdn.net/qice675563721/article/details/79781384)  
