---
layout: post 
title: '初试Tencent_AILab_ChineseEmbedding'
date: 2021-03-03
categories: AI
tags: AI
---

初试Tencent_AILab_ChineseEmbedding

### 前言

> 官网：https://ai.tencent.com/ailab/nlp/en/embedding.html

描述为：A corpus on continuous distributed representations of Chinese words and phrases.

也就是ai实验室基于中文训练的一套词向量

腾讯云官网开放词向量查找，qps限制比较死。

### 使用

网上使用遇到最多的问题就是加载缓慢，github有热心群众针对不同需求拆分了下

> https://github.com/cliuxinxin/TX-WORD2VEC-SMALL

使用python gensim库加载

```python
from gensim.models import KeyedVectors
from time import time

tic= time()
file = '' # 解压出来的 Tencent_AILab_ChineseEmbedding.txt 的路径
wv_from_text = KeyedVectors.load_word2vec_format(file, binary=False) 
toc = time()
print(toc - tic)
```

耗时25分钟左右，当然可以load bin文件，效果会好一点

#### Nd4j使用

> https://stackoverflow.com/questions/43633092/is-it-possible-to-use-gensim-word2vec-model-in-deeplearning4j-word2vec

```python
   w2v_model.wv.save_word2vec_format("path/to/w2v_model.bin", binary=True)
```

转存bin文件后续Load速度直接加快

```java
   Word2Vec w2vModel = WordVectorSerializer.readWord2VecModel("path/to/w2v_model.bin");
```

转存后使用java dl4j库load即可

```xml
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-core</artifactId>
            <version>1.0.0-beta7</version>
        </dependency>
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-nlp</artifactId>
            <version>1.0.0-beta7</version>
        </dependency>
        <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native-platform</artifactId>
            <version>1.0.0-beta7</version>
        </dependency>
```

使用java ap readWord2VecModel 800w词向量直接卡住，放弃了

#### Go+annoy server部署

> https://github.com/huichen/wordvector_be

参考github部署方案，docker化了下

对go语言不是很熟悉，打包该工程dockerfile代码如下

```
FROM golang:alpine

# 配置annoyindex环境
RUN apk add swig && apk add gcc && apk add g++
ENV GOPATH="/go"
ENV GOROOT="/usr/local/go"
COPY annoy-master/ $GOROOT/annoyindex/

WORKDIR $GOROOT/annoyindex/
RUN swig -go -intgosize 64 -cgo -c++ src/annoygomodule.i && mkdir -p $GOROOT/src/annoyindex 
RUN cp src/annoygomodule_wrap.cxx src/annoyindex.go src/annoygomodule.h src/annoylib.h src/kissrandom.h test/annoy_test.go $GOROOT/src/annoyindex
```

docker run 打包后的镜像，curl测试结果成功

```
~ liming$ curl --location --request GET 'http://localhost:8081/get.similar.keywords/?keyword=美白&num=20'
{"keywords":[{"word":"美白","similarity":1},{"word":"淡斑","similarity":0.8916605},{"word":"美白产品","similarity":0.8722978},{"word":"美白效果","similarity":0.8654123},{"word":"想美白","similarity":0.86464494},{"word":"美白祛斑","similarity":0.8604745},{"word":"能美白","similarity":0.8593319},{"word":"美白功效","similarity":0.85687655},{"word":"美白护肤","similarity":0.85603666},{"word":"美白保湿","similarity":0.8557893},{"word":"祛斑","similarity":0.85431975},{"word":"祛痘","similarity":0.8531519},{"word":"保湿美白","similarity":0.8519845},{"word":"补水美白","similarity":0.84696996},{"word":"改善肤质","similarity":0.8451075},{"word":"美白补水","similarity":0.8444052},{"word":"祛黄","similarity":0.8426262},{"word":"皮肤美白","similarity":0.8399938},{"word":"淡斑美白","similarity":0.8388319},{"word":"具有美白","similarity":0.8384946}]}
```

性能尚未测试，按照作者压测效果满足现有业务诉求