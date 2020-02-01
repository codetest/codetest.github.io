# Tensorflow求导
在网上发现一个比较好的tensorflow和deeplearning的[学习资源](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book)。在现在学习能力是很重要的，于是跟着学，跟着写代码。当然学习需要文档，可以参看[API文档](https://tensorflow.google.cn/api_docs/)（没有被墙）。求导的文旦可以参看[求导](https://tensorflow.google.cn/api_docs/python/tf/GradientTape)

## 一元函数
比如我们有一个函数 y = a * x^5 + b * x^2 + c，有基础的都可以求出导数函数为y' = 5 * a * x^4 + 2 * b * x。 对于x在2出的导数值为80a + 4b。
我们参照其中的一个[求导例子](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book/blob/master/ch01/autograd.py)来写一遍。
```pythonimport tensorflow as tf
a = tf.constant(2.)
b = tf.constant(3.)
c = tf.constant(4.)
x = tf.constant(2.)
with tf.GradientTape() as tape: # 构建函数
  tape.watch(x) # 将x视作变量，类似tf.Variable
  y = a * x** 5 + b * x ** 2 + c # 建立表达式
  dy_dx = tape.gradient(y, x) # 得到输出
  print(dy_dx)

# 得到输出如下
# tf.Tensor(172.0, shape=(), dtype=float32)
# 80 * 2 + 4 * 3 = 172
```
同时我们也会疑问，这只是求一阶导数，如果求二阶导数呢？ 比如上面表达式的二阶导数为y'' = 20 * a * x ^ 3 + 2 * b，在x = 2的二阶导数值为20 * 2 * 2^3 + 2 * 3
```pythonimport tensorflow as tf
import tensorflow as tf
a = tf.constant(2.)
b = tf.constant(3.)
c = tf.constant(4.)
x = tf.constant(2.)
with tf.GradientTape(persistent = True) as tape: # 构建函数，persistent = True是为了多次执行gradient函数
  tape.watch(x) # 将x视作变量
  y = a * x** 5 + b * x ** 2 + c # 建立表达式
  dy_dx = tape.gradient(y, x) # 得到输出
  dy_dx_dx = tape.gradient(dy_dx, x)
  print(dy_dx_dx)
  print(dy_dx)

# 得到输出如下
# tf.Tensor(326.0, shape=(), dtype=float32) 
# tf.Tensor(172.0, shape=(), dtype=float32)
```

## 多元函数求导
比如我们有一个三元函数 z = 5 * x^7 - 9 * y^4，那么得到在x = 2, y = 3处的值为 dz/dx = 35 * x^6 = 2240, dz/dy = -36 * y^3 = -972
实现如下
```pythonimport tensorflow as tf
import tensorflow as tf
x = tf.constant(2.)
y = tf.constant(3.)
with tf.GradientTape(persistent = True) as tape: # 构建函数
  tape.watch([x,y]) # 将x, y视作变量
  z = 5 * x ** 7 - 9 * y ** 4 # 建立表达式
  dz_dx = tape.gradient(target=z, sources=[x]) # 得到输出
  dz_dy = tape.gradient(target=z, sources=[y]) # 得到输出
  print(dz_dx)
  print(dz_dy)

# 得到输出如下
# [<tf.Tensor: id=21, shape=(), dtype=float32, numpy=2240.0>]
# [<tf.Tensor: id=42, shape=(), dtype=float32, numpy=-972.0>]
```
具体的求导函数可以参考[文档](https://tensorflow.google.cn/api_docs/python/tf/GradientTape#gradient)

[回到主页](https://codetest.github.io)
