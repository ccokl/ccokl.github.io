> 保证一个类只有一个实例

```javascript
class SingleClass {
  constructor(x,y) {
    this.x = x
    this.y = y
    // 判断是否已经new过1个实例
    if (!SingleClass.instance) {
      // 若这个唯一的实例不存在，那么先创建它
      SingleClass.instance = this
    }
    // 如果这个唯一的实例已经存在，则直接返回
    return SingleClass.instance
  }
}
const s1 = new SingleClass(1,2)
const s2 =  new SingleClass(2,3)
s1 === s2 // true
```

闭包实现

```javascript
function SingleClass() {}
SingleClass.getInstance = (function(){
  let instance = null 
  return function(){
    if(!instance) {
      instance = new SingleClass()
    }
    return instance
  }
})()
const s3 = SingleClass.getInstance()
const s4 = SingleClass.getInstance()
s3 === s4 // true
```

Vue插件中单例的实现

Vue规定每个插件必须实现一个install函数，Vue.use使用插件时默认执行插件的install函数，一般情况下在一个单页项目中插件只需要全局定义一次就好，就像VueX是管理整个应用的数据流，如果多次注册应该默认只返回已经注册过的，当然vue.use已经做了一层过滤

```javascript
export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }
    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this) // 去掉plugin参数将Vue对象作为install第一个参数
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
}
```

VueX 中的单例模式（确保Store的唯一性）

```javascript
let Vue // bind on install
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```

