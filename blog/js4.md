```javascript
console.log( a ); 
var a = 2;
```

执行输出`undefined`

```javascript
a = 2;
var a; 
console.log( a );
```

执行输出`2`

说明：`javaScript` 运行时在编译器阶段会最先处理`var a; `也就是变量被提升

```javascript
foo();
function foo() {
    console.log( a ); // undefined var a = 2;
    var a = 2;
}
```

执行输出`undefined`，执行`foo();`时找到了函数的声明，但实际代码写在`foo()`的后面。

说明：函数声明会被提升，函数作用域里变量也会被提升

```javascript
foo(); 
var foo = function bar() { 
    console.log(a)
    var a =2
};
```

执行输出`TypeError: foo is not a function`

说明：函数表达式不会被提升，`foo()` 由于对 `undefined`进行调用抛出`TypeError`异常

```javascript
var foo;
foo(); // 1
foo = function() { 
    console.log( 2 );
};
function foo() {
     console.log( 1 );
}

```

执行输出`1`

说明：虽然函数和变量都能被提升，但是函数声明提升优先于变量声明提升

```javascript
foo(); // 3
function foo() { 
    console.log( 1 );
}
var foo = function() { 
    console.log( 2 );
};
function foo() { 
    console.log( 3 );
}
```

执行输出`3`,后面的函数声明会覆盖前面的函数声明

总结：var 变量声明和函数声明其实跟`javaScript`运行原理有关，在`javaScript`运行之前会先进行编译，在编译阶段会先为变量和函数根据作用域规则开辟内存空间并初始化变量为`undefined`,而函数是初始化函数变量并且直接赋值函数体，以便在执行的时候方便查找和存储。

**let ， const到底有没有有变量提升？**

```javascript
console.log(a)
let a=2
```

输出：`ReferenceError: Cannot access 'a' before initialization`，表明变量没有被定义，不能在变量为初始化之前使用。其实从这句话看来`js`是找到了变量`a`的,但`a`还没有被初始化；这样看来，`let`其实也有变量提升（在我理解上来说，提升就是先为变量创造内存空间），`js` 处理`let`声明变量的过程：

```javascript

{
  let x = 1
  x = 2
}
```

1. 查找`let`声明的变量`x`，并在作用域中创建变量，不进行初始化
2. 执行代码`x=1`，初始化为1
3. 执行`x = 2`赋值为2

`let` 特性：

```javaScript
let a=2
let a=3
console.log(a)
```

输出：`SyntaxError: Identifier 'a' has already been declared`,不能重复定义变量

**参考资料：**

《你不知道的javaScript 上卷》第四章 变量提升

[let深入理解---let存在变量提升吗]: https://blog.csdn.net/jolab/article/details/82466362

MDN：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let