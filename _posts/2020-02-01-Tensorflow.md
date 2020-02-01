# Tensorflow学习
在网上发现一个比较好的tensorflow和deeplearning的[学习资源](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book)。在现在学习能力是很重要的，于是跟着学，跟着写代码。

## 求导数
比如我们有一个函数 y = a * x^5 + b * x^2 + c，有基础的都可以求出导数函数为y' = 5 * a * x^4 + 2 * b * x。 对于x在2出的导数值为80a + 4b。
我们参照其中的一个[求导例子](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book/blob/master/ch01/autograd.py)来写一遍。
```pythonimport tensorflow as tf
a = tf.constant(2.)
b = tf.constant(3.)
c = tf.constant(4.)
x = tf.constant(2.)
with tf.GradientTape() as tape: # 构建函数
  tape.watch([x]) # 将x视作变量
  y = a * x** 5 + b * x ** 2 + c # 建立表达式
  [dy_dx] = tape.gradient(y, [x]) # 得到输出
  print(dy_dx)

# 得到输出如下
# tf.Tensor(172.0, shape=(), dtype=float32)
# 80 * 2 + 4 * 3 = 172
```
