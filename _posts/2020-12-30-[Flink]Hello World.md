---
layout: post 
title: '[Flink]Hello World'
date: 2020-12-30
categories: Flink
tags: Flink
---

### install

> 详情建议参考
>
> https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/try-flink/local_installation.html

### 源码

> https://github.com/apache/flink

例子推荐WordCount.jar，看看里面做了什么

```
flink/flink-examples/flink-examples-streaming/src/main/java/org/apache/flink/streaming/examples/wordcount/WordCount.java 
```

默认数据为，也支持文本路径输入，结果数据输出。

```java
public static final String[] WORDS =
        new String[] {
            "To be, or not to be,--that is the question:--",
            "Whether 'tis nobler in the mind to suffer",
            "The slings and arrows of outrageous fortune",
            "Or to take arms against a sea of troubles,",
            "And by opposing end them?--To die,--to sleep,--",
            "No more; and by a sleep to say we end",
            "The heartache, and the thousand natural shocks",
            "That flesh is heir to,--'tis a consummation",
            "Devoutly to be wish'd. To die,--to sleep;--",
            "To sleep! perchance to dream:--ay, there's the rub;",
            "For in that sleep of death what dreams may come,",
            "When we have shuffled off this mortal coil,",
            "Must give us pause: there's the respect",
            "That makes calamity of so long life;",
            "For who would bear the whips and scorns of time,",
            "The oppressor's wrong, the proud man's contumely,",
            "The pangs of despis'd love, the law's delay,",
            "The insolence of office, and the spurns",
            "That patient merit of the unworthy takes,",
            "When he himself might his quietus make",
            "With a bare bodkin? who would these fardels bear,",
            "To grunt and sweat under a weary life,",
            "But that the dread of something after death,--",
            "The undiscover'd country, from whose bourn",
            "No traveller returns,--puzzles the will,",
            "And makes us rather bear those ills we have",
            "Than fly to others that we know not of?",
            "Thus conscience does make cowards of us all;",
            "And thus the native hue of resolution",
            "Is sicklied o'er with the pale cast of thought;",
            "And enterprises of great pith and moment,",
            "With this regard, their currents turn awry,",
            "And lose the name of action.--Soft you now!",
            "The fair Ophelia!--Nymph, in thy orisons",
            "Be all my sins remember'd."
        };
```

输出为

```shell
[root@TENCENT64 ~/flink/flink-1.12.0]# more log/flink-*-taskexecutor-*.out
(to,1)
(be,1)
(or,1)
(not,1)
(to,2)
(be,2)
(that,1)
(is,1)
(the,1)
(question,1)
(whether,1)
(tis,1)
(nobler,1)
(in,1)
(the,2)
(mind,1)
(to,3)
(suffer,1)
(the,3)
```

核心就是这一句话

```java
DataStream<Tuple2<String, Integer>> counts =
        // split up the lines in pairs (2-tuples) containing: (word,1)
        text.flatMap(new Tokenizer())
                // group by the tuple field "0" and sum up tuple field "1"
                .keyBy(value -> value.f0)
                .sum(1);
```

sum up tuple field "1"，这里就是出现次数

和spark程序很像~

### 参考

[2020 Flink 学习路线总结（持续更新](https://zhuanlan.zhihu.com/p/105620944)