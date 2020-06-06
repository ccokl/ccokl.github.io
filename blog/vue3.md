初始化

```javascript
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
```

对 `watch` 对象做遍历，拿到每一个 `handler`，`Vue` 支持 `watch` 的同一个 `key` 对应多个 `handler`，所以如果 `handler` 是一个数组，则遍历这个数组，调用 `createWatcher` 方法，否则直接调用 `createWatcher`：

```javascript
function createWatcher (
  vm: Component,
  expOrFn: string | Function,
  handler: any,
  options?: Object
) {
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {
    handler = vm[handler]
  }
  return vm.$watch(expOrFn, handler, options)
}
```

对 `hanlder` 的类型做判断，拿到它最终的回调函数，最后调用 `vm.$watch(keyOrFn, handler, options)` 函数，`$watch` 是 `Vue` 原型上的方法

```javascript
Vue.prototype.$watch = function (
    expOrFn,
    cb,
    options
  ) {
    var vm = this;
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {};
    options.user = true;
    var watcher = new Watcher(vm, expOrFn, cb, options);
    if (options.immediate) {
      try {
        cb.call(vm, watcher.value);
      } catch (error) {
        handleError(error, vm, ("callback for immediate watcher \"" + (watcher.expression) + "\""));
      }
    }
    return function unwatchFn () {
      watcher.teardown();
    }
  };
```

 `$watch` 方法是用户可以直接调用的，它可以传递一个对象，也可以传递函数,先判断 `cb` 如果是一个对象，则调用 `createWatcher` 方法，接着执行 `const watcher = new Watcher(vm, expOrFn, cb, options)` 实例化了一个 `watcher`，如果设置了 `immediate` 为 true，则直接会执行回调函数 `cb`。最后返回了一个 `unwatchFn` 方法，它会调用 `teardown` 方法去移除这个 `watcher`。

```javascript
get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }
```

后来看了`Watcher`构造函数的逻辑想了半天`watch`监听的属性是如何关联当前`Watcher`的，原来在`get`函数里`value = this.getter.call(vm, vm)`这句代码巧妙的收集了依赖，在`watch`初始化之前`data`已经初始化完毕，`this.getter.call(vm, vm)`对于`watch`的监听器来说实质上是获取当前监听的属性，在获取属性时可以收集依赖

至于watch的使用用法可以参考：[Vue.js中 watch 的高级用法](https://juejin.im/post/5ae91fa76fb9a07aa7677543)

源码分析参考：[watch](https://ustbhuangyi.github.io/vue-analysis/v2/reactive/computed-watcher.html#watch)