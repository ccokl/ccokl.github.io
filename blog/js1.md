#### 一、类型与值

**1.0  javaScript 七种内置类型:**

- 空值(null)
- 未定义(undefined)
- 布尔值( boolean)
- 数字(number)
- 字符串(string)
- 对象(object)
- 符号(symbol，ES6 中新增)

**1.1 typeof** 运算符查看值的类型

`typeof` 对`null`的处理显示“object”，并非显示“null”，`typeof null === "object"; // true`这是`javaScript`语言的一个`bug`，需要使用复合条件来检测` null` 值的类型

```javascript
var a = null;
(!a && typeof a === "object"); // true
```

`function`不是` JavaScript` 的一个内置类型，它是 `object `的一个“子类型”,但是`typeof` 处理函数显示`"function"`

`typeof function a(){ /* .. */ } === "function"; // true`

对于数组类型，也是 object 的一个“子类型”`typeof [1,2,3] === "object"; // true`

**1.2 undefined 和 undeclared**

```javascript
var a;
typeof a; // "undefined" 
typeof b; // "undefined"
```

 b 是 一个undeclared变量，但typeof b并没有报错,因为typeof有一个特殊的安全防范机制。利用typeof的安全防范机制，可以安全的判断，即使变量不存在也不会报错阻断程序如：

```javaScript
// 如果DEBUG不存在js引擎就会报错
if (DEBUG) {
	console.log( "Debugging is starting" ); 
}
// 这样是安全的
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" ); 
}
```

**1.3 数组**

在 JavaScript 中，数组可以容纳任何类型的值，可以是字符串、 数字、对象(object)，甚至是其他数组

```javaScript
var a = [ 1, "2", [3] ];
a.length; // 3 
a[0] === 1; // true 
a[2][0] === 3; // true
```

数组声明后即可向其中加入值，不需要预先设定大小

```javascript
var a = [ ]; 
a.length; // 0
a[0] = 1; 
a[1] = "2";
a[2] = [ 3 ];
a.length; // 3
```

含有空白或空缺单元的数组，可以正常运行

```javascript
var a = [ ];
a[0] = 1;
// 此处没有设置a[1]单元 
a[2] = [ 3 ];
a[1]; // undefined 
a.length; // 3
```

可以包含字符串键值和属性，但这些并不计算在数组长度内

```javascript
var a = [ ];
a[0] = 1; 
a["foobar"] = 2;
a.length; // 1
a["foobar"];  // 2
a.foobar; // 2

```

如果字符串键值能够被强制类型转换为十进制数字的话，就会被当作数字索引来处理。

```javascript
var a = [ ]; 
a["13"] = 42; 
a.length; // 14
```

类数组转换成数组，ES5中可以通过数组的工具函数如`Array.prototype.slice.call() `  ES6工具函数 Array.from(..)

**1.4 字符串**

字符串和数组很类似，有length 属性以及 indexOf(..)和 concat(..) 方法，但javaScript 中字符串是**不可变的**，字符串的成员函数不会改变其原始值，而是创建并返回一个新的字符串。可以通过“借用”数组的**非变更方**法来处理字符串:

```javascript
var a = "foo"
var c = Array.prototype.join.call(a,"-")
var d = Array.prototype.map.call(a,(v)=>{
    return v.toUpperCase() + '.'
}).join("")
```

`join`数组原型对象上的方法，可以将数组转换成字符串,

```javascript
var a = "foo"
Array.prototype.reverse.call(a)
console.log(a)
```

在字符串上使用数组的反转函数会报错，因为字符串一旦初始化就不会改变`TypeError: Cannot assign to read only property '0' of object '[object String]'`

**1.5 数字**

`tofixed(..) `方法可指定小数部分的显示位数

```javaScript
var a = 42.59;
a.toFixed( 0 ); // "43" 
a.toFixed( 1 ); // "42.6" 
a.toFixed( 2 ); // "42.59" 
a.toFixed( 3 ); // "42.590" 
a.toFixed( 4 ); // "42.5900"
```

`toPrecision(..) `方法用来指定有效数位的显示位数:

```javascript
var a = 42.59;
a.toPrecision(1 ); // "4e+1"
a.toPrecision( 2 ); // "43"
a.toPrecision( 3 ); // "42.6"
```

上面的方法不仅适用于数字变量，也适用于数字常量

```javascript
// 无效语法,. 被视为常量 42. 的一部分
42.toFixed( 3 ); // SyntaxError
// 下面的语法都有效: 
(42).toFixed( 3 ); // "42.000" 
0.42.toFixed( 3 ); // "0.420" 
42..toFixed( 3 ); // "42.000"
//42..tofixed(3) 则没有问题，因为第一个 . 被视为 number 的一部分，第二个 . 是属性访问 运算符
42 .toFixed(3); // "42.000"
```

从 ES6 开始，严格模式(strict mode)不再支持 0363 八进制格式(新格式如 下)。0363 格式在非严格模式(non-strict mode)中仍然受支持

```javaScript
ES6 支持以下新格式:
0o363; // 243的八进制
0O363; // 同上 
0b11110011; // 243的二进制
0B11110011; // 同上
```

`0.1 + 0.2 === 0.3; // false`二进制浮点数中的 0.1 和 0.2 并不是十分精确，它们相加的结果并非刚好等于 0.3，而是一个比较接近的数字 0.30000000000000004，所以条件判断结果为 false。

**怎样来判断 0.1 + 0.2 和 0.3 是否相等呢?**

设置一个误差范围值，通常称为“机器精度”(machine epsilon)，对`javaScript` 的数字来说，这个值通常是 `2^-52 (2.220446049250313e-16)`。

从 ES6 开始，该值定义在` Number.EPSILON` 中

```javascript
if (!Number.EPSILON) {
	Number.EPSILON = Math.pow(2,-52);
}
```

```javascript
使用 Number.EPSILON 来比较两个数字是否相等
function numbersCloseEnoughToEqual(n1,n2) {
  return Math.abs( n1 - n2 ) < Number.EPSILON;
}
var a = 0.1 + 0.2; 
var b = 0.3;
numbersCloseEnoughToEqual( a, b ); numbersCloseEnoughToEqual( 0.0000001, 0.0000002 ); // false
```

`Number. MAX_VALUE` :最大浮点数大约是 1.798e+308

`Number.MIN_VALUE`:最小浮点数大约是 5e-324,无限接近于 0 

**整数的安全范围**

ES6:` Number.MAX_SAFE_INTEGER:`“安全”呈现的最大整数是2^53 - 1，即9007199254740991

ES6:` Number. MIN_SAFE_INTEGER`:最 小 整 数 是 -9007199254740991

由于 JavaScript 的数字类型无法精确呈现 64 位数值，所以必须将它们保存(转换)为字符串

**整数检测**

ES6 中的 `Number.isInteger(..) `:检测一个值是否是整数

polyfill Number.isInteger(..) 方法

```javascript
if (!Number.isInteger) { 
  Number.isInteger = function(num) {
		return typeof num == "number" && num % 1 == 0; 
  };
}
```

ES6 中的 Number.isSafeInteger(..) 方法:检测一个值是否是安全的整数

polyfill Number.isSafeInteger(..) 方法:

```javascript
if (!Number.isSafeInteger) { 
  Number.isSafeInteger = function(num) {
  return Number.isInteger( num ) &&
    Math.abs( num ) <= Number.MAX_SAFE_INTEGER;
  }; 
}
```

**32 位有符号整数**

a | 0可以将变量a中的数值转换为32位有符号整数，因为数位运算符|只适用于32位 整数,因此与 0 进行操作即可截取 a 中 的 32 位数位。

**1.6 特殊数值**

**undefined**

- 指从未赋值
- 是一个标识符，可以被当作变量来使用和赋值
- void 运算符,undefined 是一个内置标识符(除非被重新定义，见前面的介绍)，它的值为 undefined， 通过 void 运算符即可得到该值。

```javascript
var a = 42;
console.log( void a, a ); // undefined 42
//void并不改变表达式的结果， 只是让表达式不返回值:
```

**null**

- null 指空值(empty value)
- 曾赋过值，但是目前没有值
- null 是一个特殊关键字，不是标识符

**NaN**

- 不是数字的数字
- 是一个“警戒值”(sentinel value，有特殊用途的常规值)，用于指出数字类型中的错误 情况，即“执行数学运算没有成功，这是失败后返回的结果”。
- NaN是一个特殊值，它和自身不相等，是唯一一个非自反

```javascript
var a = 2 / "foo"; 
var b = "foo";
a; // NaN b; "foo"
window.isNaN( a ); // true
window.isNaN( b );// true

```

isNaN(..) 检查参数是否不是 NaN，也不是数字,"foo" 不是一个数字，但是它也不是 NaN

ES6工具函数 Number.isNaN(..)

```javascript
if (!Number.isNaN) { 
  Number.isNaN = function(n) {
  return (
    typeof n === "number" && window.isNaN( n )
 }
 var a = 2 / "foo"
 var b ="foo"
); };
Number.isNaN( a ); // true
Number.isNaN( b );// false
```

**Infinity**

```javascript
var a = 1 / 0; // Infinity 
var b = -1 / 0; // -Infinity
```

**零值**

为什么会有 -0 ？

有些应用程序中的数据需要以级数形式来表示(比如动画帧的移动速度)，数字的符号位 (sign)用来代表其他信息(比如移动的方向)。此时如果一个值为 0 的变量失去了它的符号位，它的方向信息就会丢失。所以保留 0 值的符号位可以防止这类情况发生。

区分 -0 和 0方法：

```javascript
 function isNegZero(n) {
    n = Number( n );
		return (n === 0) && (1 / n === -Infinity); 
 }
isNegZero( -0 ); // true
isNegZero( 0 / -3 ); // true
isNegZero( 0 );// false
```

**特殊等式**

Es6工具方法 Object.is(..) 

```javascript
var a = 2 / "foo"; 
var b = -3 * 0;
Object.is( a, NaN ); 
Object.is( b, -0 );
Object.is( b, 0 );
```

```javascript
if (!Object.is) {
Object.is = function(v1, v2) {
  // 判断是否是-0
  if (v1 === 0 && v2 === 0) {
      return 1 / v1 === 1 / v2;
  }
  // 判断是否是NaN 
  if (v1 !== v1) {
       return v2 !== v2;
   }
   // 其他情况
   return v1 === v2;
 };
}
```

**1.7 值和引用**

```javascript
var a = 2;
var b = a; // b是a的值的一个副本 
b++;
a; // 2
b; // 3
var c = [1,2,3];
var d = c; // d是[1,2,3]的一个引用 
d.push( 4 );
c; // [1,2,3,4]
d; // [1,2,3,4]
```

简单值(即标量基本类型值，scalar primitive)总是通过值复制的方式来赋值 / 传递，包括 null、undefined、字符串、数字、布尔和 ES6 中的 symbol。

复合值(compound value)——对象和函数，则总 是通过引用复制的方式来赋值 / 传递。

引用指向的是值本身而非变量，所以一个引用无法更改另一个引用的指向

```javascript
var a = [1,2,3];
var b = a;
a; // [1,2,3]
b; // [1,2,3]
// 然后
b = [4,5,6]; 
a; // [1,2,3] 
b; // [4,5,6]
```

b=[4,5,6] 并不影响 a 指向值 [1,2,3]，除非 b 不是指向数组的引用，而是指向 a 的指针，在 javaScript 中不存在指针

```javascript
function foo(x) { 
   x.push( 4 );
   x; // [1,2,3,4]
  // 然后
   x = [4,5,6]; 
   x.push( 7 );
	 x; // [4,5,6,7]
}
var a = [1,2,3];
foo( a );
a; // 是[1,2,3,4]，不是[4,5,6,7]
```

向函数传递 a 的时候，实际是将引用 a 的一个复本赋值给 x，而 a 仍然指向 [1,2,3]。 在函数中我们可以通过引用x来更改数组的值(push(4)之后变为[1,2,3,4])。但x = [4,5,6] 并不影响 a 的指向，所以 a 仍然指向 [1,2,3,4]。不能通过引用 x 来更改引用 a 的指向，只能更改 a 和 x 共同指向的值。

`foo( a.slice() );`
 slice(..) 不带参数会返回当前数组的一个浅复本(shallow copy)。由于传递给函数的是指向该复本的引用，所以 foo(..) 中的操作不会影响 a 指向的数组。

如果要将标量基本类型值传递到函数内并进行更改，就需要将该值封装到一个复合值(对象、数组等)中，然后通过引用复制的方式传递。

```javascript
function foo(wrapper) {
	wrapper.a = 42;
}
var obj = { 
	a: 2
};
foo( obj ); 
obj.a; // 42
```