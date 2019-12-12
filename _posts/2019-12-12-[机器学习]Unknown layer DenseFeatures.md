---
layout: post 
title: 'Unknown layer DenseFeatures'
date: 2019-12-12
categories: 机器学习
tags: 机器学习
---

### 前言

可以说遇到一个血坑

https://www.tensorflow.org/tutorials/structured_data/feature_columns

使用官网的demo训练好模型后，进行模型导出，然后使用model.save()，结果model.load()一直报错

```
Unknown layer DenseFeatures
```

stackoverflow求解

https://github.com/tensorflow/tensorflow/issues/27008

https://github.com/tensorflow/tensorflow/issues/34189

得到的结论更多是

> same problem
>
> The script executes successfully if you save/load the model in `tf` format.
>
> ValueError: Unknown layer: DenseFeatures
>
> is there any solutions now ? I am also facing the same problem

### 解决方案

```python
tf.keras.models.save_model(model, '/path/',save_format='tf')
```

```python
model = tf.keras.models.load_model("/Users/liming/Desktop/data/predict")
```

亲测可用，用默认的model.save就会出现这个问题，并且导出的问题没有assets这些文件。