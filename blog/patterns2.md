> 观察者模式定义了一种一对多的依赖关系，，让多个观察者对象同时监听某一个目标对象，当这个目标对象的状态发生变化时，会通知所有观察者对象，使它们能够自动更新。

Vue响应式系统实现原理

<img src="/Users/xusenyue/Downloads/未命名文件.png" alt="未命名文件" style="zoom:200%;" />

observer：是数据监听器也是发布者

Dep：用于收集订阅者和通知订阅者

watcher：真实的订阅者，收到数据更新去更新视图

complie：编译器，负责指令解析，创建订阅者等

```javaScript
// 简单模拟只考虑对象
class Dep {
  static target = null
  constructor(){
    // 初始化订阅队列
    this.subs = []
  }
  // 添加订阅者
  addSub(sub) {
    this.subs.push(sub)
  }
  // 给当前目标对象添加订阅者
  depend() {
    console.log("添加订阅者")
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
  // 通知订阅者
  notify() {
    console.log("通知更新")
    console.log(this.subs)
    this.subs.forEach(sub=>{
      sub.update()
    })
  }
}
function Observer(data) {
  if(data && typeof data === 'object') {
    Object.keys(data).forEach(key=>{
      // 给目标属性添加上数据监听器
      defineReactive(data,key,data[key])
    })
  }
}
function defineReactive(data,key,val) {
  // 如果属性值也是对象，递归属性值给加上监听器
  Observer(val)
  const dep = new Dep()
  Object.defineProperty(data,key,{
    get:function(val){
      // 绑定订阅者
      if (Dep.target) {
       dep.depend()
      }
      return val
    },
    set:function(value){
      // 通知订阅者
      dep.notify()
      val = value
    }
  })
}
class Watcher {
  constructor(){
     this.get()
  }
  // 获取当前监听器
  get () {
    console.log("创建订阅者")
    Dep.target = this
  }
  // 添加
  addDep(dep){
    dep.addSub(this)
    console.log(dep,'添加依赖')
  }
  // 更新操作
  update(){
    console.log('更新视图')
  }
}
let a = {b:1}
Observer(a) // 初始化响应式数据
new Watcher()
a.b // 触发依赖收集
a.b = 2 // 触发更新机制

// 创建订阅者
// 添加订阅者
// Dep { subs: [ Watcher {} ] } 添加依赖
// 通知更新
// [ Watcher {} ]
// 更新视图

```





