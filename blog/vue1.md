<img src="https://cn.vuejs.org/images/data.png" alt="data" style="zoom:50%;" />

**1、[Object.defineProperty](https://cn.vuejs.org/v2/guide/reactivity.html)**

`Vue2.x` 使用`Object.defineProperty` 将 `Vue` 实例中的`data`对象全部转为[getter/setter](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_Objects#定义_getters_与_setters)。在内部让 `Vue` 能够追踪依赖,在属性被访问和修改时通知变更。

```javascript
Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend() // 收集依赖项
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify() //通知watcher更新组件
    }
  })
```

**2、Dep Class**

`Dep `主要做了两件事：收集依赖以及通知`watcher`更新

```javascript
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }
  // 依赖收集
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
  // 通知更新
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```

**3、Watcher  Class**

`Watcher ` 是一个中间管理器，可以**添加依赖以及更新操作等**,`Vue`在挂载模板是时会注册一个`Render Watcher`。

```javascript
 var updateComponent = function () {
   vm._update(vm._vnode, false);
  };
  new Watcher(vm, updateComponent, noop, null, true);
```

`addDep`用于收集依赖，从`Vue`源码来看在添加依赖的过程中对于同一个依赖做了过滤处理，也就是同一个依赖只添加一次

```javascript
addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }
```

`update`用于更新操作，

```javascript
update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }
```

对于更新操作`Vue`有标识识别是否进行同步更新，如果不是，会先存放在一个队列里，在`queueWatcher`函数里会先对`watcher`去重

```javascript
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      nextTick(flushSchedulerQueue) //异步执行队列
    }
  }
}
```

**4、总结**

- ` Vue` 会在初始化实例时对属性执行 `getter/setter` 转化，所以属性必须在 `data` 对象上存在才能让` Vue` 将它转换为响应式的。
- `Vue` 不允许动态添加根级响应式属性，所以必须在初始化实例前声明所有根级响应式属性。
- `Vue`挂载模板时会注册一个`Render Watcher`
- `Vue` 在更新 `DOM` 时是**异步**执行的。

