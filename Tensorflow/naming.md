[TOC]

# 命名
Tf里面很重要的一点就是命名，包括operation、tensor、variable、constant等等。下面说一下一些有关命名的小知识。
1. api一般都提供了name参数让用户自行决定要使用的类别
```Python
import tensorflow as tf

a = tf.constant(1.0)
b = tf.constant(2.0, name='b')

print(a.name)  # 'Const:0'
print(b.name)  # 'b:0'
```

2. 如果提供了的名字已经被使用了，那么tf会在名字右面加个后缀
```Python
import tensorflow as tf

a = tf.constant(1.0, name='a')
aa = tf.constant(2.0, name='a')

print(a.name)  # 'a:0'
print(aa.name) # 'a_1:0'
```

3. 那么问题来了，名字里不同的数字都有哪些含义呢？
  为了理解清楚这些数字的含义，首先我们要区分操作(operation)和张量(tensor)的关系和区别：操作产生张量。我们把`a = tf.constant(1.0, name='a')`分成左右两部分来看：简单地就可以理解为左边是一个张量，而右边则是一个操作，所以我们实质上都是在调用操作来获取需要的张量。回到原来的话题，`'a:0'`里的冒号前面的字符串`'a'`就是得到该张量的操作名，而`'a:0'`则是这个张量的名称（也是唯一标识符）。我们可以通过张量的`op`属性来知道是哪种操作产生该张量的。

```Python
import tensorflow as tf

a = tf.constant(1.0, name='a')

print(a.name)    # 'a:0'
print(a.op.name) # 'a'
```
所以，我们在tf设置`name=`的时候实质上是给操作命名，而tf则会为我们自动产生相应的张量名。你可能会问，那
为毛还要加个冒号和数字呢，直接用操作名不好？这是因为有些操作会产生多个张量输出，这样我们就需要不同的
数字来区分不同的张量了，举个例子：
```Python
import tensorflow as tf

a, b = tf.nn.top_k([1], 1)

print(a.name)  # 'TOPV2_1:0'
print(b.name)  # 'TOPV2_1:1'
```
所以，总结起来就是：`'a_1:0'`中，冒号前面的1是为了区分不同的操作，而冒号后面的0则是为了区分不同的张量。
