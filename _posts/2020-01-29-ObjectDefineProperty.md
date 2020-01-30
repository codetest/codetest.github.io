# Object.defineProperty

在2.x版本的[vue](https://github.com/vuejs/vue)中，它使用Object.dfineProperty来检测数据变化，从而达到数据驱动视图的效果。以下根据[MDN的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)做一些简要的介绍（还是读英文版的好些）。

## 简单使用
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

## 函数定义
以下是函数的定义接口
![Definition](/images/defineProperty/Definition.png)

从接口来看，第一个参数就是被操作的对象，第二个参数是key或者Symbol，第三个有些复杂。针对这种情况，建议通过typescript查看定义。因为TypeScript是强类型的，可以看到参数的具体类型，以下是从TypeScript摘抄过来的代码片段
```typescript
    /**
     * Adds a property to an object, or modifies attributes of an existing property.
     * @param o Object on which to add or modify the property. This can be a native JavaScript object (that is, a user-defined object or a built in object) or a DOM object.
     * @param p The property name.
     * @param attributes Descriptor for the property. It can be for a data property or an accessor property.
     */
    defineProperty(o: any, p: PropertyKey, attributes: PropertyDescriptor & ThisType<any>): any;
```
PropertyKey定义
```typescript
declare type PropertyKey = string | number | symbol;
```
PropertyDescriptor定义
```typescript
interface PropertyDescriptor {
    configurable?: boolean;
    enumerable?: boolean;
    value?: any;
    writable?: boolean;
    get?(): any;
    set?(v: any): void;
}
```
ThisType定义
```typescript
/**
 * Marker for contextual 'this' type
 */
interface ThisType<T> { }
```

根据MDN的文档，Descriptor分为Data Descriptor和Access Descriptor。两者共同拥有configurable（propety是否可以被删除，重新修改？）和enumerable（for循环中可否出现）属性。Data Descriptor拥有: value和writable属性。Access Descriptor拥有get和set两个函数。两者的冲突关系处理如下。
```typescript
/*
If a descriptor has neither of value, writable, get and set keys, it is treated as a data descriptor. If a descriptor has both value or writable and get or set keys, an exception is thrown.
*/
```
如果configurable没有定义，那么对应的属性不能出行定义或者删除，以下是一个例子。
![Definition1](/images/defineProperty/Definition1.png)
如果configurable为true，那么问题就可以得到解决。

## 检测数据变化
vue中当数据发生变化时，需要变化视图。那么就需要知道，什么时候发生变化，和通知谁更新视图。那么我们就可以通过Access Descriptor达到这个目的。以下是一个实例。
```typescript
function defineReactive(obj, key, val){
	Object.defineProperty(obj, key, {
		configurable: true,
		enummerable: true,
		get: () => {
            // 此处收集依赖
			console.log("get value", val)
			return val;
		},
		set: (newVal) => {
            // 此处通知更新变化
			console.log("new value", newVal)
			val = newVal;
		}
	})
}

var obj = {a: 5}
defineReactive(obj, "a", obj["a"])
obj.a
obj.a = 9
obj.a

//输出如下
/*
get value 5
new value 9
get value 9
*/
```
然而，在vue当中并没有那么简单，如果判定obj[key]是一个Object，那么就需要递归下去。在对object赋值的时候也要重新做监控，之前的监控就要去掉。果是一个Array就需要监控数组相关操作，元素变化就不能检测了。

## vue 中的实际使用
我们通过以下代码来调试以下vue具体怎么实现数据驱动变化的。
```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<div id="app">
{{message}}
</div>
<script type="text/javascript">
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
</script>
```
通过打断点，我们得到以下的调用堆栈。
![Callstack](/images/defineProperty/Callstack.png)
由此我们可以看到，vue是在初始化的时候就对data进行defineProperty。以下我们看一下是怎么样进行依赖收集的，在get函数打一个断点。
![Callstack1](/images/defineProperty/Callstack1.png)
从堆栈看到，在装载和渲染视图的时候就获取了一下message字段。然后我们在看下其中的一下匿名函数。
![Callstack2](/images/defineProperty/Callstack2.png)
在这里面的匿名函数，调用了message字段。上面的_c就是createElement（创建vnode用的）。我们看一下这个匿名函数是怎么生成的。
![Callstack3](/images/defineProperty/Callstatck3.png)
它是在解析模板的时候，将模板编译为一个函数。里面具体再看的的时候还可以看到有对函数进行缓存的做法。解析之后得到函数的字符串形式。
```javascript
"with(this){return _c('div',{attrs:{"id":"app"}},[_v("\n"+_s(message)+"\n")])}"
```
后面通过new Function的方法实现一个函数返回设定为render.里面的_s和_v的意义有下面定义
```javascript
  /*  */

  function installRenderHelpers (target) {
    target._o = markOnce;
    target._n = toNumber;
    target._s = toString;
    target._l = renderList;
    target._t = renderSlot;
    target._q = looseEqual;
    target._i = looseIndexOf;
    target._m = renderStatic;
    target._f = resolveFilter;
    target._k = checkKeyCodes;
    target._b = bindObjectProps;
    target._v = createTextVNode;
    target._e = createEmptyVNode;
    target._u = resolveScopedSlots;
    target._g = bindObjectListeners;
    target._d = bindDynamicKeys;
    target._p = prependModifier;
  }
```
这些函数在初始化的时候已经绑定到Vue的prototype里面了。
![Callstack4](/images/defineProperty/Callstack4.png)
在这里顺便修改一下模板的结构
```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<div id="app">
<div>
{{message}}
</div>
<div>
{{message}}
</div>
</div>
<script type="text/javascript">
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
</script>
```
会响应更新生成以下的函数内容。
```javascript
"with(this){return _c('div',{attrs:{"id":"app"}},[_c('div',[_v("\n"+_s(message)+"\n")]),_v(" "),_c('div',[_v("\n"+_s(message)+"\n")])])}"
```
这个ast非常厉害~
