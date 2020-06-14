## Vuex 阅读

##### vuex全局注入:`Vue.use(Vuex);`

`install` 函数执行`Vue.mixin({ beforeCreate: vuexInit })`

```javascript
function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (__DEV__) {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue) // -> Vue.mixin({ beforeCreate: vuexInit })
}
```

`Vue2.0`以上版本混入`Vue.mixin({ beforeCreate: vuexInit })`

`vuexInit` 函数设置`this.$store`

```javascript
function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```

##### Vuex 响应式状态管理

```javascript
store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
})
```

##### Vuex 变更数据规范

Mutation —> 同步变更状态,统一变更数据出口

Action —> 	可执行异步操作

##### 深拷贝工具函数

```javascript
/**
 * Deep copy the given object considering circular structure.
 * This function caches all nested objects and its copies.
 * If it detects circular structure, use cached copy to avoid infinite loop.
 *
 * @param {*} obj
 * @param {Array<Object>} cache
 * @return {*}
 */
export function deepCopy (obj, cache = []) {
  // just return if obj is immutable value
  if (obj === null || typeof obj !== 'object') {
    return obj
  }

  // if obj is hit, it is in circular structure
  // 破解循环引用
  const hit = find(cache, c => c.original === obj)
  if (hit) {
    return hit.copy
  }

  const copy = Array.isArray(obj) ? [] : {}
  // put the copy into cache at first
  // because we want to refer it in recursive deepCopy
  cache.push({
    original: obj,
    copy
  })

  Object.keys(obj).forEach(key => {
    copy[key] = deepCopy(obj[key], cache)
  })

  return copy
}
```

