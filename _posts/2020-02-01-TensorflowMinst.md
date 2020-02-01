# Tensorflow读MINST数据集建模
具体的建模代码看[这里](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book/blob/master/ch03/main.py) 然而里面只是提供了console形式的建模过程。 以下对这个程序进行扩展，能够做实际的识别工作。具体的运行细节就不解释了，都是高级接口，都弄好了。

## 评估测试数据
从[代码](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book/blob/master/ch03/main.py)来看，一直在计算的是这个3层的线性神经网络。当我们得到model里面所有变量的值的时候，就可以对其他输入进行计算。
对代码里面的数据集来看，我们用前100个数据进行计算，首先是将x进行reshape，然后得到的out里面选择最大值的索引，比如索引为0就是识别为1，1识别为2，以此类推。
```python
    import numpy as np
    correct = 0
    for i in range(100):
        test_x = tf.reshape(x[i], (-1, 28*28))
        test_y = y[i]
        out = model(test_x)
        expect = (np.argmax(test_y.numpy()))
        actual = (np.argmax((out[0]).numpy()))
        if (expect == actual):
            correct = correct + 1
    print(correct)
 ```
 这个代码在我本机跑得倒的correct是95，其实还是可以的。
 
 ## 测试真实图片
 上面的x是按照下面的处理经过归一化的。加入有一张满足28 * 28大小的图片，怎么进行识别？
```python
x = tf.convert_to_tensor(x, dtype=tf.float32) / 255.
```
