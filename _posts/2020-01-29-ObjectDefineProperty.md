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

根据MDN的文档，Descriptor分为Data Descriptor和Access Descriptor。两者共同拥有configurable（propety是否可以被删除）和enumerable（for循环中可否出现）属性。Data Descriptor拥有: value和writable属性。Access Descriptor拥有get和set两个函数。两者的冲突关系处理如下。
```typescript
/*
If a descriptor has neither of value, writable, get and set keys, it is treated as a data descriptor. If a descriptor has both value or writable and get or set keys, an exception is thrown.
*/
```