---
layout: post 
title: '数据可视化-pyplot'
date: 2019-12-03
categories: 机器学习
tags: 机器学习
---

#### 前言

加入深度学习的坑，很多时候就是参数调优，免不了的就是图形可视化

#### matplotlib.pyplot

利用python的图形库，简单进行准确率和loss的可视化，观察每个epoch训练的曲线变化情况，以便评估模型效果。

```python
import matplotlib.pyplot as plt
```

```python
history = model.fit(train_ds,
          validation_data=val_ds,
          epochs=10)
```

这里，训练的时候我指定了训练集和验证集，epochs设置为10，方便观察图像

```
print(history.history.keys())
```

用到的数据key可以通过该函数打印

```python
plt.plot(history.history['accuracy'])
plt.plot(history.history['val_accuracy'])
plt.title('model accuracy')
plt.ylabel('accuracy')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left') 
plt.show()
# summarize history for loss 
plt.plot(history.history['loss']) 
plt.plot(history.history['val_loss']) 
plt.title('model loss')
plt.ylabel('loss')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left') 
plt.show()
```

我在fit过程中，也相应打印了数值以及key值

```
Epoch 1/10
159/159 [==============================] - 117s 738ms/step - loss: 0.3706 - accuracy: 0.8578 - val_loss: 0.0000e+00 - val_accuracy: 0.0000e+00
Epoch 2/10
159/159 [==============================] - 149s 936ms/step - loss: 0.3649 - accuracy: 0.8578 - val_loss: 0.3469 - val_accuracy: 0.8624
Epoch 3/10
159/159 [==============================] - 188s 1s/step - loss: 0.3617 - accuracy: 0.8579 - val_loss: 0.3480 - val_accuracy: 0.8619
Epoch 4/10
159/159 [==============================] - 211s 1s/step - loss: 0.3592 - accuracy: 0.8589 - val_loss: 0.3453 - val_accuracy: 0.8624
Epoch 5/10
159/159 [==============================] - 241s 2s/step - loss: 0.3576 - accuracy: 0.8586 - val_loss: 0.3452 - val_accuracy: 0.8626
Epoch 6/10
159/159 [==============================] - 265s 2s/step - loss: 0.3557 - accuracy: 0.8590 - val_loss: 0.3443 - val_accuracy: 0.8631
Epoch 7/10
159/159 [==============================] - 320s 2s/step - loss: 0.3539 - accuracy: 0.8590 - val_loss: 0.3436 - val_accuracy: 0.8627
Epoch 8/10
159/159 [==============================] - 356s 2s/step - loss: 0.3537 - accuracy: 0.8586 - val_loss: 0.3440 - val_accuracy: 0.8624
Epoch 9/10
158/159 [============================>.] - ETA: 2s - loss: 0.3533 - accuracy: 0.8589
```

然后就是简单的图形可视化了，后续研究tensorboard，该工具更强大，然后在总结总结。

![img](/images/posts/machine/2_1.png)

效果很差，后面优化~
