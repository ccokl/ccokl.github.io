> 原型模式就是拷贝出一个新对象

 javaScript中的创建对象和继承都是基于原型设计的，所以对于javaScript而言原型模式更像一种编程范式

ES6 的类其实是原型继承的语法糖:

```javascript
class Animal {
  constructor(name ,age) {
   this.name = name
   this.age = age
  }
  
  eat() {
    console.log(吃')
  }
}
```

其实完全等价于写了这么一个构造函数:

```javascript
function Animal(name, age) {
  this.name = name
  this.age = age
}

Dog.prototype.eat = function() {
  console.log('吃')
}
var cat = new Animal('cat',1)
```

在 javaScript 中，每个构造函数都拥有一个`prototype`属性，它指向构造函数的原型对象，这个原型对象中有一个 construtor 属性指回构造函数；每个实例都有一个`__proto__`属性，当我们使用构造函数去创建实例时，实例的`__proto__`属性就会指向构造函数的原型对象。

![a](/Users/xusenyue/Downloads/a.png)

javaScript 实现原型模式（实现JS中的深拷贝）

1、`JSON.stringify`-对象是一个严格的 JSON 对象时，可以顺利使用这个方法

2、递归实现深拷贝

```javaScript
// 不考虑数组，set, map, weakset, weakmap……
function isObject(x) {
    return Object.prototype.toString.call(x) === '[object Object]';
}
function clone(source){
  if(isObject(source)) return source
  let target = {}
  for(let key in source){
    if(source.hasOwnProperty(key)){
        target[key] = clone(source[key])
    }
  }
  return target
}
```

3、循环实现深拷贝

```javascript
// 不考虑数组，set, map, weakset, weakmap……
function isObject(x) {
    return Object.prototype.toString.call(x) === '[object Object]';
}
function cloneLoop(obj) {
	let root ={}
	let loopList = [
		{
			parent:root,
			key:null,
			data:obj
		}
	]
  while(loopList.length > 0) {
    let node = loopList.pop()
    let parent= node.parent
    let key = node.key
    let data = node.data
    let res = parent
    // 如果有key挂载到parent，并初始化
    if(key) {
      res = parent[key]={}
    }
    for(let k in data) {
      if(data.hasOwnProperty(k)) {
        if(isObject(data[k])){
          loopList.push({
            parent:res,
            key:k,
            data:data[k]
          })
        }else {
          res[k] = data[k]
        }
      }
    }
  }
  return root 
}
```

![截屏2020-04-06下午10.16.54](/Users/xusenyue/Desktop/截屏2020-04-06下午10.16.54.png)

[参考-深拷贝的终极探索](https://yanhaijing.com/javascript/2018/10/10/clone-deep/)

[jQuery中的extend方法源码](https://github.com/jquery/jquery/blob/1472290917f17af05e98007136096784f9051fab/src/core.js#L121)

