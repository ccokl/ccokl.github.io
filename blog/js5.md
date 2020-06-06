#### 什么是闭包

闭包是javaScript语言的一种特性，在 javaScript 中以函数作为承接单元。当函数可以`记住并访问`所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。

```javascript
function foo() {
  var a = 2;
	function bar() { 
    console.log( a ); // 2
	}
	bar(); 
}
foo();
```

`bar()` 产生了闭包，但是由于它内嵌在`foo`函数里面执行，`bar`中对`a`的引用遵循了`词法作用域的查找规则`，其实也是闭包查询规则的一部分。

```javascript
function foo() {
  var a = 2;
	function bar() { 
    console.log( a ); // 2
	}
	return bar; 
}
(foo())();
```

稍微修改一下上面的代码，将`bar`  `return` 出来，在`foo`函数外执行，`bar`依旧可以获取到`a`,而这时候`bar`它记住了`foo`的内部词法作用域，并且可以按照词法作用域查询规则找到`a`。由此可想`bar`能够引用`foo`的内部作用域，说明`foo`的内部作用域并没有被回收，而正因为闭包的存在，引擎没有回收`foo`的内部作用域的内存空间。

#### 回调函数

```javascript

function foo() { 
  var a = 2;
  function baz() { 
    console.log( a ); // 2
  }
	bar( baz ); 
}
function bar(fn) {
	fn(); // fn是一个闭包!
}
```

对函数类型的值进行传递，会形成一个闭包，所以常见的回调函数采用了闭包，如：

```javascript
function wait(message) {
	setTimeout( function timer() { console.log( message );
}, 1000 ); }
wait( "Hello, closure!" );
```

`setTimeout`中的内部函数`timer`是一个具有 `wait`的作用域的闭包。

在定时器、事件监听器、 Ajax 请求、跨窗口通信、Web Workers 或者任何其他的异步(或者同步)任务中，只要使 用了回调函数，实际上就是在使用闭包。

#### 模块

```javascript
function CoolModule() {
  var something = "cool";
  var another = [1, 2, 3];
  function doSomething() { 
    console.log( something );
  }
  function doAnother() {
  	console.log( another.join( " ! " ) );
  }
  return {
    doSomething: doSomething, 
    doAnother: doAnother
  }; 
}
var foo = CoolModule(); 
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

`CoolModule()`通过return 出一个对象，包含其两个内部函数，而两个内部函数是包含着`CoolModule`的内部作用域的闭包。

模块的特点

- 为创建内部作用域而调用了一个包装函数;
- 装函数的返回 值必须至少包括一个对内部函数的引用，这样就会创建涵盖整个包装函数内部作用域的 闭包。

单例模块

```javascript
var foo = (
  function CoolModule() { 
    var something = "cool";
		var another = [1, 2, 3];
    function doSomething() { 
      console.log( something );
    }
    function doAnother() {
      console.log( another.join( " ! " ) );
    }
    return {	
      doSomething: doSomething, 
      doAnother: doAnother
    }; 
  })();
foo.doSomething(); // cool 
foo.doAnother(); // 1 ! 2 ! 3
```

通过内部函数修改内部私有变量的值

```javascript
var foo = (function CoolModule(id) { 
  function change() {
	// 修改公共 API
	publicAPI.identify = identify2; }
  function identify1() { 
    console.log( id );
  }
  function identify2() {
    console.log( id.toUpperCase() );
  }
  var publicAPI = {
    	change: change,
      identify: identify1
   };
	return publicAPI; 
})( "foo module" );
  foo.identify(); // foo module foo.change();
  foo.identify(); // FOO MODULE
```

#### 模块依赖加载器 

```javascript
var MyModules = (
  function Manager() {
	var modules = {};
	function define(name, deps, impl) {
    for (var i=0; i<deps.length; i++) {
      deps[i] = modules[deps[i]];
    }
    modules[name] = impl.apply( impl, deps ); 
  }
  function get(name) { 
    return modules[name];
  }
  return {
    define: define,
    get: get 
  };
})();
```

使用

```javascript
MyModules.define( "bar", [], function() { 
    function hello(who) {
      return "Let me introduce: " + who;
    }
    return {
    	hello: hello
    }; 
} );
MyModules.define( "foo", ["bar"], function(bar) {
  var hungry = "hippo";
  function awesome() {
    console.log( bar.hello( hungry ).toUpperCase() );
  }
  return {
    awesome: awesome
  }; 
} );
var bar = MyModules.get( "bar" ); 
var foo = MyModules.get( "foo" );
console.log(
bar.hello( "hippo" )
); // Let me introduce: hippo 
foo.awesome(); // LET ME INTRODUCE: HIPPO
```

ES6 模块机制

```javascript
// bar.js
function hello(who) {
	return "Let me introduce: " + who;
}
export hello; 

//foo.js
// 仅从 "bar" 模块导入 hello() 
import hello from "bar";
var hungry = "hippo";
function awesome() { 
  console.log(
	hello( hungry ).toUpperCase() );
}
export awesome;

//baz.js
module foo from "foo"; 
module bar from "bar";
console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino 
foo.awesome(); // LET ME INTRODUCE: HIPPO
```



#### 经典的循环

```javascript
for (var i=1; i<=5; i++) { 
  setTimeout( function timer() {
		console.log( i ); }, i*1000 );
}
```

以上会依次输出 6，6，6，6，6，6，因为 `setTimeout`中的闭包会在循环结束后依次执行，而每一个闭包所引用的作用域都是一致的，`i`变量在全局作用域中。如果想要输出我们想要的结果可以加入更多的闭包如：

```javascript
for (var i=1; i<=5; i++) { 
  (function(j) {
		setTimeout( 
    	function timer() { console.log( j );
      }, i*1000 );
   })(i);
}
```

将`i`作为变量传入闭包函数中，每个闭包函数中通过隐试的创建了自己的作用域变量`j`

更加简单的方法使用let

```javascript
for (let i=1; i<=5; i++) { 
  setTimeout( function timer() {
		console.log( i ); }, i*1000 );
}

```

`let` 可以创建块级作用域，这样每一个循环都是一个作用域块,每个作用域互不干扰。
