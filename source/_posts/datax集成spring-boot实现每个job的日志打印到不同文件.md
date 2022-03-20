---
title: datax集成spring boot后实现每个job打印到不同文件
date: 2021-06-14 11:29:35
tags: datax
---
### 问题背景

1.当我们的程序开启了多个线程进行作业的时候，如何把跟一个作业相关的日志都打印到一个文件呢，而不是都打印在一个文件里，毕竟都打印在一个文件里，很不方便我们日常去查看问题。

2.经过调研发现slfj的MDC就可以实现这个功能，原理就是通过threalocal去实现的，感兴趣的可以去查看源码。

### 解决方法

1.具体怎么使用也很简单，我们需要在datax的线程池执行任务的时候，能够获取到启动这个线程池传递过来的jobId就可以了，具体代码如下。
```java
package com.alibaba.datax.core.util;

import org.slf4j.MDC;

import java.util.Map;
import java.util.concurrent.*;

public class MDCThreadPoolExecutor extends ThreadPoolExecutor {

    public MDCThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    public MDCThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
    }

    public MDCThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, handler);
    }

    public MDCThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }

    @Override
    public void execute(Runnable command) {
        final Map<String, String> context = MDC.getCopyOfContextMap();
        super.execute(new Runnable() {
            @Override
            public void run() {
                // 将父线程的MDC内容传给子线程
                MDC.setContextMap(context);
                try {
                    // 执行异步操作
                    command.run();
                } finally {
                    // 清空MDC内容
                    MDC.clear();
                }
            }
        });
    }
}

```

```java
package com.alibaba.datax.core.util;

import org.slf4j.MDC;

import java.util.Map;

public class MDCRunnable implements Runnable {
    private Runnable runnable;
    /**
     * 保存当前主线程的MDC值
     */
    private final Map<String, String> mainMdcMap;

    public MDCRunnable(Runnable runnable) {
        this.runnable = runnable;
        this.mainMdcMap = MDC.getCopyOfContextMap();
    }


    @Override
    public void run() {
        // 将父线程的MDC值赋给子线程
        for (Map.Entry<String, String> entry : mainMdcMap.entrySet()) {
            MDC.put(entry.getKey(), entry.getValue());
        }
        // 执行被装饰的线程run方法
        runnable.run();
        // 执行结束移除MDC值
        for (Map.Entry<String, String> entry : mainMdcMap.entrySet()) {
            MDC.put(entry.getKey(), entry.getValue());
        }
    }
}

```
通过上述对线程池进行改造，在线程池每次获取任务的时候，都可以从父线程获取父线程设置的变量。因此我们要在父线程启动子线程的时候，设置上这个变量。
```java
MDC.put("jobId", String.valueOf(jobId));

```

接下来，我们需要配置下logback.xml
```xml

    <appender name="siftInfo" class="ch.qos.logback.classic.sift.SiftingAppender">
        <!--discriminator鉴别器，设置运行时动态属性,siftingAppender根据这个属性来输出日志到不同文件 -->
        <discriminator>
            <key>jobId</key>
            <defaultValue>unknown</defaultValue>
        </discriminator>
        <sift>
            <!--具体的写日志appender，每一个userId创建一个文件-->
            <appender name="${jobId}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                <append>true</append>
                <encoder charset="UTF-8">
                    <pattern>%d{yyyy-MM-dd HH:mm:ss} %level %logger{36} - %msg%n</pattern>
                </encoder>

                <file>${logdir}/job/${jobId}.log</file>
                <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                    <!--定义文件滚动时的文件名的格式-->
                    <fileNamePattern>./logs/job/%d{yyyyMMdd}/${jobId}-%i.log</fileNamePattern>
                    <maxFileSize>500MB</maxFileSize>
                    <maxHistory>60</maxHistory>
                    <totalSizeCap>20GB</totalSizeCap>
                </rollingPolicy>

                <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
                    <level>INFO</level>
                </filter>
            </appender>
        </sift>
    </appender>

```
