# vue computed解析

vue的computed函数有缓存效果，先看一下这个怎么实现的， 缓存什么时候更新的。

## computed的初始化
初始化的时候会先覆盖computed中的key的get进行重新定义。
```javascript
function initComputed$1 (Comp) {
  var computed = Comp.options.computed;
  for (var key in computed) {
    defineComputed(Comp.prototype, key, computed[key]);
  }
}
```
这时候的get定义的时候，_computedWatchers还没有创建。
```javascript
function defineComputed (
  target,
  key,
  userDef
) {
  var shouldCache = !isServerRendering();
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : createGetterInvoker(userDef);
    sharedPropertyDefinition.set = noop;
  } else {
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : createGetterInvoker(userDef.get)
      : noop;
    sharedPropertyDefinition.set = userDef.set || noop;
  }
  if (false) {}
  Object.defineProperty(target, key, sharedPropertyDefinition);
}

function createComputedGetter (key) {
  return function computedGetter () {
    var watcher = this._computedWatchers && this._computedWatchers[key];
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate();
      }
      if (Dep.target) {
        watcher.depend();
      }
      return watcher.value
    }
  }
}
```

后面就会实质地初始化watchers，而watcher的get会调用computed的get。
```javascript
function initComputed (vm, computed) {
  // $flow-disable-line
  var watchers = vm._computedWatchers = Object.create(null);
  // computed properties are just getters during SSR
  var isSSR = isServerRendering();

  for (var key in computed) {
    var userDef = computed[key];
    var getter = typeof userDef === 'function' ? userDef : userDef.get;
    if (false) {}

    if (!isSSR) {
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      );
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef);
    } else if (false) {}
  }
}
```
创建watcher的时候，lazy值赋给了dirty。对于cmoputed而言，它第一次被get的话，会被evalute一遍。
```javascript
  this.dirty = this.lazy; // for lazy watchers
```

当computed的第一次被get的时候，对应的data在get的时候会把这个watcher添加到data的dependencies，当data有变化的时候，watcher只会标记dirty为true。在nextTick的时候，重新渲染，调用get函数重新获取数据。

