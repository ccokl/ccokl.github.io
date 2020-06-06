```javascript
function foo() {
    console.log( a );
}
function bar() { 
    var a = 3;
    foo(); 
}
var a = 2; 
bar();
```

上面这段代码为什么会输出2，而不是3？

`javaScript` 只有词法作用域，`foo()`的定义在全局作用域，执行时会在它所在词法作用域查找变量`a`

#### `this` 的作用域

```javascript
function foo() {
  var a = 2;
	this.bar(); 
  console.log(this)  
}
function bar() { 
  console.log( this.a );
}
foo(); // ReferenceError: a is not defined
```

`说明 this`并不指向`foo`函数的词法作用域,在非严格模式的node环境下打印` console.log(this)  ` ,发现`this`指向`global` 全局对象

```javascript
Object [global] {
  global: [Circular],
  clearInterval: [Function: clearInterval],
  clearTimeout: [Function: clearTimeout],
  setInterval: [Function: setInterval],
  setTimeout: [Function: setTimeout] { [Symbol(util.promisify.custom)]: [Function] },
  queueMicrotask: [Function: queueMicrotask],
  clearImmediate: [Function: clearImmediate],
  setImmediate: [Function: setImmediate] {
[Symbol(util.promisify.custom)]: [Function]
  }
}
```

而在严格模式下`this` 是`undefined `

```javascript
'use strict'
function foo() {
    var a = 2;
   console.log(this)  //undefined          
 }
 foo();
```

由此可以猜测`this` 可以用来绑定函数执行时的`上下文`，所谓上下文就是函数调用所要的各种环境变量等，在非严格模式下`node` 的全局上下文是`global` 而浏览器是`window`，`this` 不遵循词法作用域，而是作用在函数执行的时候。

#### `this` 绑定规则

**1、默认绑定**,`this` 指向全局对象

```javascript
var a = 2;
function foo() {
	bar();  
}
function bar() { 
  console.log( this.a );
}
foo(); // 2
```

**2、隐试绑定**，`this` 指向`obj`对象

```javascript
function foo() { 
  console.log( this.a );
}
var obj = { 
  a: 2,
	foo: foo 
};
obj.foo(); // 2
```

隐试绑定丢失

```javascript
function foo() { 
  console.log( this.a );
}
var obj = { 
  a: 2,
	foo: foo 
};
var bar = obj.foo; 
var a = "oops, global"; // a 是全局对象的属性 
bar(); // "oops, global"
```

`foo` 并不属于`obj`对象，将`obj.foo; ` 赋值给`bar`变量，实际上这是一个声明在全局的表达式，这时调用`bar()`, `his` 在非严格模式下默认指向全局对象

**3、显示绑定**（在某个对象上强制调用函数）

`call`在调用 `foo` 时强制把它的 `this `绑定到 `obj` 上

```javascript
function foo() {
  console.log( this.a );
}
var obj = { 
  a:2
};
foo.call( obj ); // 2
```

硬绑定

```javascript
function foo(something) { 
  console.log( this.a, something ); 
  return this.a + something;
}
var obj = { 
  a:2
};
var bar = function() {
	return foo.apply( obj, arguments );
};
var b = bar( 3 ); // 2 3 
console.log( b ); // 5
bar.call(window,3) // 2 3（硬绑定的 bar 不可能再修改它的 this）
```

辅助函数

```javascript
function foo(something) { 
  console.log( this.a, something ); 
  return this.a + something;
 }
// 简单的辅助绑定函数 
function bind(fn, obj) {
  return function() { 
    return fn.apply(obj, arguments );
  }; 
 }
var obj = { 
  a:2
};
var bar = bind( foo, obj );
var b = bar( 3 ); // 2 3 
console.log( b ); // 5

```

`Function.prototype.bind` ES5 内置的硬绑定函数

```javascript
function foo(something) { 
  console.log( this.a, something ); 
  return this.a + something;
}
var obj = { 
  a:2
};
var bar = foo.bind( obj );
var b = bar( 3 ); // 2 3 console.log( b ); // 5
```

`foo.bind( obj )` 返回一个硬绑定`this`的函数

 （1）创建一个全新的对象。
 （2） 这个新对象会被执行[[Prototype]]连接。
 （3）这个新对象会绑定到函数调用的this。
 （4）如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

```javascript
function foo(a) { 
	this.a = a;
}
var bar = new foo(2);
console.log( bar.a ); // 2
```

#### `this` 绑定的优先级

```javascript
function foo() { 
	console.log( this.a );
}
var obj1 = { 
  a: 2,
  foo: foo 
};
var obj2 = { 
  a: 3,
  foo: foo
};
obj1.foo(); // 2 
obj2.foo(); // 3
obj1.foo.call( obj2 ); // 3 
obj2.foo.call( obj1 ); // 2
```

`obj1.foo.call( obj2 );`输出3，说明绑定的是obj2，显式绑定>隐试绑定

```javascript
function foo(something) { 
  this.a = something;
}
var obj1 = { 
  foo: foo
};
var obj2 = {};
obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 ); 
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 ); 
console.log( obj1.a ); // 2 
console.log( bar.a ); // 4

```

`console.log( obj1.a ); // 2 ` 说明`new obj1.foo( 4 )` 没有先操作隐试绑定，new 绑定 > 隐式绑定

```javascript
function bind(fn, obj) { 
    return function() {
        fn.apply( obj, arguments ); 
    };
}
function foo(something) { 
	this.a = something;
}
var obj1 = {};
var bar = bind(foo,obj1 );
bar( 2 );
console.log( obj1.a ); // 2
var baz = new bar(3); 
console.log( obj1.a ); // 2 
console.log( baz.a ); // undefined
```

`new` 绑定不能修改显式绑定后函数的`this`绑定

但是使用ES内置`bind`的属性，`console.log( baz.a );` 输出3，说明`new` 绑定成功，new 绑定 >显式绑定

```javascript
function foo(something) { 
	this.a = something;
}
var obj1 = {};
var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2
var baz = new bar(3); 
console.log( obj1.a ); // 2 
console.log( baz.a ); // 3
```

实现 ES5中的`bind`

```javascript
if (!Function.prototype.bind) {
    Function.prototype.bind = function (oThis) {
      // bind 关在函数的原型链上
        if (typeof this !== "function") { 
            throw new TypeError(
                "Function.prototype.bind - what is trying " + "to be bound is not callable"
            );
        }
      // 获取调用函数的参数
        var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, // 将当前this保存起来
        fNOP = function () { }, 
        fBound = function () {
          	// 当前对象是否在fNOP原型链上，是将this指向当前对象，不是将this指向oThis
            return fToBind.apply((this instanceof fNOP && oThis ? this : oThis),
            aArgs.concat(Array.prototype.slice.call(arguments)))
        }
        fNOP.prototype = this.prototype; // 继承this的原型
        fBound.prototype = new fNOP(); // 将fBound指向fNOP的实例
        return fBound;
    };
}
```

 因为`return fToBind.apply((this instanceof fNOP && oThis ? this : oThis),`这段代码判断了如果new 一个bind硬绑定的函数，将新生成的优先将生成的对象绑定到`this`上

**软绑定实现**，保留隐试绑定和显示绑定

```javascript
if (!Function.prototype.softBind) {
    Function.prototype.softBind = function (obj) {
        var fn = this;
        var curried = [].slice.call(arguments, 1); 
        var bound = function () {
            return fn.apply(
                (!this || this === (window || global)) ?
                    obj : this,
                curried.concat.apply(curried, arguments)
            );
        };
        bound.prototype = Object.create(fn.prototype);
        return bound;
    };
}
```

`!this || this === (window || global)) ? obj : this,` 在绑定对象为全局或者找不到时，采用显示绑定，然则采用隐试绑定。

#### 更安全的`this`

```javascript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}
// 我们的 DMZ 空对象
var ø = Object.create( null ); // 把数组展开成参数
foo.apply( ø, [2, 3] ); // a:2, b:3
// 使用 bind(..) 进行柯里化 
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

将`this` 绑定到空对象减少副作用，如果传入`null` ，`this`默认会绑定全局对象，这样会修改全局对象

#### ES6中的箭头函数

```javascript
function foo() { 
	setTimeout(() => {
    // 这里的 this 在词法上继承自 foo()
    console.log( this.a ); 
  },100);
}
var obj = {
	a:2
};
foo.call( obj ); // 2
```

箭头函数的内部`this`继承自其外部函数的`this`作用域。