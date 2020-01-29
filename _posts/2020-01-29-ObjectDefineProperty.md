#Object.defineProperty

在2.x版本的[vue](https://github.com/vuejs/vue)中，它使用Object.dfineProperty来检测数据变化，从而达到数据驱动视图的效果。以下根据[MDN的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)做一些简要的介绍（还是读英文版的好些）。

##主要功能
The static method Object.defineProperty() defines a new property directly on an object, or modifies an existing property on an object, and returns the object.这个静态方法可以在一个对象上面定义一个新的property或者修改已存在的property，然后返回值是这个对象。
以下是一个简单的函数使用
```js
const obj = {a: 5}
var ret = Object.defineProperty(obj, 'a', {value: 4})
ret;
obj;
```
输出效果为
![Demo1](/images/defineProperty/Demo1.png)
但是为了修改一个属性，这么做貌似有些小题大做。
