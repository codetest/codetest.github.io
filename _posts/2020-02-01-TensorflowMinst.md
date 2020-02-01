# Tensorflow对MINST数据集建模
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
 这个代码在我本机跑得倒的correct是96，其实还是可以的。
 
## 测试真实图片
[readMNIST](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book/blob/master/ch03/readMNIST.py)提供了抽取测试图片的方法。上面的x是按照下面的处理经过归一化的。加入有一张满足28 * 28大小的图片，怎么进行识别？
```python
x = tf.convert_to_tensor(x, dtype=tf.float32) / 255.
```
而我们做灰度化就只需要按照加权平均的方法实现f(i,j)=0.30R(i,j)+0.59G(i,j)+0.11B(i,j)。
以下是转化代码
```python
from PIL import Image, ImageDraw
def loadAsNormalizedGrayedImage(file):
    # 加权平均法 + PIL 进行灰度化
    img = Image.open(file)
    pixel = img.load()  # cv2的图像读取后可以直接进行操作，而Image打开的图片需要加载
    w, h = img.size
    data = [[0 for x in range(h)] for x in range(w)]
    for i in range(img.size[0]):
        for j in range(img.size[1]):
            # Y = 0．3R + 0．59G + 0．11B
            # 通过Image格式打开的图片，像素格式为 RGB，数据需要逆转以下。
            data[j][i] = (0.3 * pixel[i, j][0] + 0.11 * pixel[i, j][2] + 0.59 * pixel[i, j][1]) / 255.    
    return data
```
根据上面的[readMNIST](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book/blob/master/ch03/readMNIST.py)数据的时候，只需要简单读值就可以了，具体代码如下
```python
from PIL import Image, ImageDraw
def loadAsNormalizedGrayedImage(file):
    # 加权平均法 + PIL 进行灰度化
    img = Image.open(file)
    pixel = img.load()  # cv2的图像读取后可以直接进行操作，而Image打开的图片需要加载
    w, h = img.size
    data = [[0 for x in range(h)] for x in range(w)]
    for i in range(img.size[0]):
        for j in range(img.size[1]):
            # Y = 0．3R + 0．59G + 0．11B
            # 通过Image格式打开的图片
            data[i][j] = pixel[j, i] / 255.
    return data
```

另外为了方便运行，需要把模型序列化。我们可以查看[Model的文档](https://tensorflow.google.cn/api_docs/python/tf/keras/Model?hl=en)可以看到有load_weights和save_weights的接口。因此在训练完模型之后可以见到调用model.save_weights("model.weights")来序列话，测试时只需要重新定义之前的网络模型，然后加载权重数据即可
```python
model = keras.Sequential([ 
    layers.Dense(512, activation='relu'),
    layers.Dense(256, activation='relu'),
    layers.Dense(10)])
model.load_weights("model.weights")
```
对于以下10个图片测试结果分别为
![img](/images/MNIST/0.png)![img](/images/MNIST/1.png)![img](/images/MNIST/2.png)![img](/images/MNIST/3.png)![img](/images/MNIST/4.png)![img](/images/MNIST/5.png)![img](/images/MNIST/6.png)![img](/images/MNIST/7.png)![img](/images/MNIST/8.png)![img](/images/MNIST/9.png)![img](/images/MNIST/10.png)
```python
7
2
1
0
4
1
4
9
2
9
0
```
自己手写用mspaint构造的测试集
![img](/images/MNIST/n0.png)![img](/images/MNIST/n1.png)![img](/images/MNIST/n2.png)![img](/images/MNIST/n3.png)![img](/images/MNIST/n4.png)![img](/images/MNIST/n5.png)![img](/images/MNIST/n6.png)![img](/images/MNIST/n7.png)![img](/images/MNIST/n8.png)![img](/images/MNIST/n9.png)
得到的结果为，只有5个识别对了，当然这个数据集有一个特点就是像素点非黑即白，线的宽度一致。另外和线性模型的有限性有关。
```python
0
6
2
3
2
5
0
2
8
3
```
完整代码如下
```python
import  tensorflow as tf
from    tensorflow import keras
from    tensorflow.keras import layers, optimizers, datasets
import numpy as np
model = keras.Sequential([ 
    layers.Dense(512, activation='relu'),
    layers.Dense(256, activation='relu'),
    layers.Dense(10)])


from PIL import Image, ImageDraw
def loadAsNormalizedGrayedImage(file):
    # 加权平均法 + PIL 进行灰度化
    img = Image.open(file)
    pixel = img.load()  # cv2的图像读取后可以直接进行操作，而Image打开的图片需要加载
    w, h = img.size
    data = [[0 for x in range(h)] for x in range(w)]
    for i in range(img.size[0]):
        for j in range(img.size[1]):
            # Y = 0．3R + 0．59G + 0．11B
            # 通过Image格式打开的图片，像素格式为 RGB
            data[j][i] = (0.3 * pixel[i, j][0] + 0.11 * pixel[i, j][2] + 0.59 * pixel[i, j][1]) / 255.    
            # data[i][j] = pixel[j, i] / 255.
    return data

if __name__ == '__main__':
    model.load_weights("model.weights")
    #model.save_weights("model.weights")
    for i in range(10):
        img = loadAsNormalizedGrayedImage("n" + str(i) + ".png")
        test_x = tf.reshape(img, (-1, 28*28))
        out = model(test_x)
        actual = (np.argmax((out[0]).numpy()))
        print(actual)
```

[回到主页](https://codetest.github.io)
