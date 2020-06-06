

#### [[Prototype]]机制

[[Prototype]]是对象内部的隐试属性，指向一个内部的链接，这个链接的作用是:如果在对象上没有找到需要的属性或者方法引用，引擎就 会继续在 [[Prototype]] 关联的对象上进行查找。同理，如果在后者中也没有找到需要的 引用就会继续查找它的 [[Prototype]]，以此类推。这一系列对象的链接被称为“原型链”。

```javascript

function Foo() { // ...
}
var a = new Foo();
Object.getPrototypeOf( a ) === Foo.prototype; // true

```

调用new Foo()时会创建对象a，其中一步就是将a内部的 [[Prototype]] 链接到 Foo.prototype 所指向的对象。new Foo()只是间接完成:一个关联到其他对象的新对象。

```javascript
function Foo() { /* .. */ }
Foo.prototype = { /* .. */ }; // 创建一个新原型对象
var a1 = new Foo();
a1.constructor === Foo; // false! 
a1.constructor === Object; // true!
```

a1 并不是由Foo构造，a1对象仅仅是委托 [[Prototype]] 链上的 Foo. prototype，在Foo. prototype并没有找到constructor，于是会在原型链上查找直到找到为止

> .constructor 并不是一个不可变属性。它是不可枚举

#### 原型继承风格

```javascript
function Foo(name) { 
  this.name = name;
}
Foo.prototype.myName = function() {
  return this.name;
};
function Bar(name,label) { 
  Foo.call( this, name ); 
  this.label = label;
}
// 创建一个新的 Bar.prototype 对象并关联到 Foo.prototype 
Bar.prototype = Object.create( Foo.prototype );
// 现在没有 Bar.prototype.constructor 了 
// 如果需要这个属性的话可能需要手动修复一下它
Bar.prototype.myLabel = function() { 
  return this.label;
};
var a = new Bar( "a", "obj a" );
a.myName(); // "a" 
a.myLabel(); // "obj a"
```

` Object.create( Foo.prototype );`会创建一个“新”对象,并把新对象内部的 [[Prototype]] 指向 Foo.prototype 

#### 对象关联设计模式

```javascript
Task = {
setID: function(ID) { this.id = ID; },
outputID: function() { console.log( this.id ); }
};
// 让XYZ委托Task
XYZ = Object.create( Task );
XYZ.prepareTask = function(ID,Label) { this.setID( ID );
this.label = Label;
};
XYZ.outputTaskDetails = function() { this.outputID();
console.log( this.label ); };
// ABC = Object.create( Task ); // ABC ... = ...
```

`XYZ = Object.create( Task );`  会返回一个新对象赋值给XYZ，这个新对象的原型指向Task，对XYZ进行操作不会改变Task，并且XYZ可以使用Task的一些方法属性，相比于原型继承模式，对象关联设计模式更为简洁

**常见错误做法：**

```javascript
Bar.prototype = Foo.prototype;
// 基本上满足，但是可能会产生一些副作用
Bar.prototype = new Foo();

```

1.`Bar.prototype = Foo.prototype` 并不会创建一个关联到 `Bar.prototype `的新对象，它只 是 让 `Bar.prototype` 直 接 引 用 `Foo.prototype `对 象。 因 此 当 你 执 行 类 似 `Bar.prototype. myLabel = ...` 的赋值语句时会直接修改 `Foo.prototype` 对象本身。

2.`Bar.prototype = new Foo() `的确会创建一个关联到 `Bar.prototype` 的新对象。但是它使用 了 `Foo(..) `的“构造函数调用”，如果函数 `Foo `有一些副作用(比如写日志、修改状态、注册到其他对象、给 this 添加数据属性，等等)，就会影响到 `Bar() `的“后代"。

```javaScript
// ES6 之前需要抛弃默认的 Bar.prototype
Bar.ptototype = Object.create( Foo.prototype );
// ES6 开始可以直接修改现有的 
Bar.prototype Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

ES6 添加了辅助函数 Object.setPrototypeOf(..)，可以用标准并且可靠的方法来修 改关联。

#### instanceof 操作符

```javascript
function Foo() { // ...
}
Foo.prototype.blah = ...;
var a = new Foo();
a instanceof Foo; // true
```

instanceof 操作符的左操作数是一个普通的对象，右操作数是一个函数。instanceof 回答

的问题是:在 a 的整条 [[Prototype]] 链中是否有指向 Foo.prototype 的对象?

#### isPrototypeOf(..) 

isPrototypeOf(..) 函数可以处理两个对象之间是否通过[[Prototype]] 链关联

`Foo.prototype.isPrototypeOf( a ); // true`

isPrototypeOf(..) 回答的问题是:在 a 的整 条 [[Prototype]] 链中是否出现过 Foo.prototype ?

#### 获取对象的原型链

获取一个对象的 [[Prototype]] 链。在 ES5 中，标准的方法是: `Object.getPrototypeOf( a );`

绝大多数浏览器也支持一种非标准的方法来访问内部 [[Prototype]] 属性:

`a.__proto__ === Foo.prototype; // true` `.__proto__ `并不存在于你正在使用的对象中 ,和其他的常用函数(.toString()、.isPrototypeOf(..)，等等)一样，存在于内置的 Object.prototype中。

`.__proto__ `的实现

```javascript
Object.defineProperty( Object.prototype, "__proto__", { 
  get: function() {
		return Object.getPrototypeOf( this ); 
  },
  set: function(o) {
  // ES6 中的 setPrototypeOf(..) 
    Object.setPrototypeOf( this, o ); 
    return o;
  } 
});
```

#### 创建对象方法 Object.create(）

```javascript
var foo = {
  something: function() {
    console.log( "Tell me something good..." ); 
  }
};
var bar = Object.create( foo ); 
bar.something(); // Tell me something good...
```

`Object.create(..) `创建一个新对象(bar)并把它关联到我们指定的对象(foo)

`Object.create(null)` 会创 建 一 个 拥 有 空( 或 者 说 null)[[Prototype]] 链接的对象

实现`Object.create(）`：

```javascript
if (!Object.create) { 
  Object.create = function(o) {
    function F(){} 
    F.prototype = o; 
    return new F();
  }; 
}
```
