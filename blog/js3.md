#### 1.1 对象内部属性 [[Class]] 

常见的原生函数：

- String()
- Number()
- Boolean()
- Array()
- Object()
- Function()
- RegExp()
- Date()
- Error()
- Symbol()——ES6

```javascript
var a = new String( "abc" );
typeof a; // 是"object"，不是"String" 
a instanceof String; // true 
Object.prototype.toString.call( a ); // "[object String]"
```

typeof 在这里返回的是对象类型的子类型。

chrome下`console.log( a );`new String("abc") 创建的是字符串 "abc" 的封装对象，而非基本类型值 "abc"。

<img src="/Users/xusenyue/Desktop/截屏2020-01-21下午4.52.06.png" alt="截屏2020-01-21下午4.52.06" style="zoom:50%;" />

`Object.prototype.toString(..) `查看对象的内部 [[Class]] 属性，通常对象的内部 [[Class]] 属性和创建该对象的内建原生构造函数相对应，但 null 和 undefined除外

```javascript
Object.prototype.toString.call( null );// "[object Null]"
Object.prototype.toString.call( undefined ); // "[object Undefined]"
Object.prototype.toString.call( "abc" ); // "[object String]"
Object.prototype.toString.call( 42 ); // "[object Number]"
Object.prototype.toString.call( true ); // "[object Boolean]"
```

Null() 和 Undefined() 这样的原生构造函数并不存在，但是内部 [[Class]] 属性值仍 然是 "Null" 和 "Undefined"；基本类型值被各自的封装对象自动包装。

#### 1.2 封装对象

基本类型值没有 `.length` 和 `.toString()` 这样的属性和方法,`javaScript` 会自动为基本类型值包装一个封装对象,浏览器为 `.length` 这样的常见情况做了性能优化，直接使用封装对象来“提前优化”代码反而会降低执行效率。

```javascript
var a = new Boolean( false );
if (!a) {
	console.log( "Oops" ); // 执行不到这里,a始终为真值
}
```

可以使用 Object(..) 函数，自行封装基本类型值

```javascript
var a = "abc";
var b = new String( a ); 
var c = Object( a );
typeof a; // "string" 
typeof b; // "object" 
typeof c; // "object"
b instanceof String; // true
c instanceof String; // true
Object.prototype.toString.call( b ); // "[object String]" Object.prototype.toString.call( c ); // "[object String]"
```

**拆封**

使用 valueOf() 函数，得到封装对象中的基本类型值

```javascript
var a = new String( "abc" ); 
var b = new Number( 42 ); 
var c = new Boolean( true );
a.valueOf(); // "abc" 
b.valueOf(); // 42 
c.valueOf(); // true
```

隐式拆封(强制类型转换)

```javascript
var a = new String( "abc" ); 
var b = a + ""; // b的值为"abc"
typeof a; // "object" 
typeof b; // "string"

```

**Symbol(..)**

- 符号是具有唯一性的特殊值(并 非绝对)，用它来命名对象属性不容易导致重名。
- 使用 Symbol(..) 原生构造函数来自定义符号不能带 new 关键字，否则会出错
- 论是在代码还是开发控制台中都无法查看和访问它的值，只会 显示为诸如 Symbol(Symbol.create) 这样的值。
- ES6 中有一些预定义符号，以 Symbol 的静态属性形式出现，如 Symbol.create、Symbol. iterator 等

#### 1.3 原生对象的原型

```javascript
typeof Function.prototype; 
Function.prototype();// "function" // 空函数!

RegExp.prototype.toString(); // "/(?:)/"——空正则表达式
"abc".match( RegExp.prototype );// [""]

Array.isArray( Array.prototype ); // true
Array.prototype.push( 1, 2, 3 );// 3
Array.prototype;// [1,2,3]
```

`Function.prototype` 是一个函数，`RegExp.prototype` 是一个正则表达式，而 `Array. prototype `是一个数组

#### 1.4 值类型转换

将值从一种类型转换为另一种类型通常称为类型转换(type casting)，这是显式的情况;隐式的情况称为强制类型转换(coercion)。

类型转换发生在静态类型语言的编译阶段，而强制类型转换则发生在动态类型语言的运行时。

```javascript
var a = 42;
var b = a + ""; // 隐式强制类型转换 
var c = String( a ); // 显式强制类型转换
```

**1.4 .1 类型转换的基本规则**

**ToString **处理非字符串到字符串的强制类型转换。

如果对象有自己的 toString() 方法，字符串化时就会调用该方法并使用其返回值。

**JSON 字符串化**

工具函数` JSON.stringify(..) `在将 `JSON `对象序列化为字符串时也用到了` toString`。

`JSON.stringify(..)` 在对象中遇到 undefined、function 和 symbol 时会自动将其忽略，在

数组中则会返回 `null`

不安全的 `JSON` 值:

- undefined
- function
- symbol
- 包含循环引用的对象

```javascript
JSON.stringify( undefined ); // undefined
JSON.stringify( function(){} );// undefined
JSON.stringify( [1,undefined,function(){},4]//"[1,null,null,4]"
); 
JSON.stringify({ a:2, b:function(){} })//"{"a":2}"
//对包含循环引用的对象执行 JSON.stringify(..) 会出错。
```

 JSON.stringify(..) 并不是强制类型转换，它涉及 ToString 强制类型转换，具体表现在以下两点：

(1) 字符串、数字、布尔值和 null 的 JSON.stringify(..) 规则与 ToString 基本相同。
 (2) 如果传递给 JSON.stringify(..) 的对象中定义了 toJSON() 方法，那么该方法会在字符串化前调用，以便将对象转换为安全的 JSON 值。



**ToNumber**处理非数字到当作数字的强制类型转换

- true 转换为 1，false 转换为 0。undefined 转换为 NaN，null 转换为 0
- 对象(包括数组)会首先被转换为相应的基本类型值，如果返回的是非数字的基本类型值，则再强制转换为数字。
- 在获取对象的基本类型时，检查该值是否有 valueOf() 方法。 如果有并且返回基本类型值，就使用该值进行强制类型转换。如果没有就使用 toString()的返回值(如果存在)来进行强制类型转换。，如果 valueOf() 和 toString() 均不返回基本类型值，会产生 TypeError 错误。
- 使用 Object.create(null) 创建的对象 [[Prototype]] 属性为 null，并且没有 valueOf() 和 toString() 方法，因此无法进行强制类型转换。

**日期显式转换为数字**

一元运算符 + 的另一个常见用途是将日期(Date)对象强制类型转换为数字，返回结果为 Unix 时间戳，以微秒为单位(从 1970 年 1 月 1 日 00:00:00 UTC 到当前时间):

```javascript
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );
+d; // 1408369986000

var timestamp = Date.now();

var timestamp = new Date().getTime();
```

**显式解析数字字符串**

```
var a = "42";
var b = "42px";
Number( a );    // 42
parseInt( a );  // 42
Number( b );    // NaN
parseInt( b );  // 42
```

解析允许字符串中含有非数字字符，解析按从左到右的顺序，如果遇到非数字字符就停 止。而转换不允许出现非数字字符，否则会失败并返回 NaN。 ES5 开始 parseInt(..) 默认转换为十进制数

```javascript
 var a = {
  num: 21,
	toString: function() { return String( this.num * 2 ); }
};
parseInt( a ); // 42
```

parseInt(..) 先将参数强制类型转换为字符串再进行解析



**ToBoolean**

1. 假值(falsy value)

   ```
   • undefined
   • null
   • false
   • +0、-0 和 NaN
   • ""
   ```

 Boolean(..) 和 !! 来进行显式转换

```javascript
var a = "0";  
var g;
!!a; // true
!!g; // false
// 不常用
Boolean( a ); // true
Boolean( g ); // false
```



**字符串和数字之间的显式转换**

```javascript
var a = 42;
var b = String( a );
var c = "3.14";
var d = Number( c );
b; // "42" 
d; // 3.14
```

字符串和数字之间的转换是通过 String(..) 和 Number(..) 来实现的，不用 new 关键字，并不创建封装对象。

```javascript
var a = 42;
var b = a.toString();
var c = "3.14"; 
var d = +c;
b; // "42" 
d; // 3.14
```

a.toString() 中涉及隐式转换。因为 toString() 对 42 这样的基本类型值不适用，所以 JavaScript 引擎会自动为 42 创建一个封 装对象然后对该对象调用 toString()

**字符串和数字之间的隐式强制类型转换**

```javascript
var a = 42;
var b = a + "";
b; // "42"
```

a + ""  会对a调用valueOf()方法，然后通过ToString抽象 操作将返回值转换为字符串。而 String(a) 则是直接调用 ToString()。

```javascript
var a = "3.14"; 
var b = a - 0;
b; // 3.14
```

-是数字减法运算符，因此a - 0会将a强制类型转换为数字。也可以使用a * 1和a /1，这两个运算符也只适用于数字

**布尔值到数字的隐式强制类型转换**

```javascript
  function onlyOne() {
    var sum = 0;
    for (var i=0; i < arguments.length; i++) { 
      // 跳过假值，和处理0一样，但是避免了NaN 
      if (arguments[i]) {
           sum += arguments[i];
      }
    }
        return sum == 1;
    }
  var a = true;
  var b = false;
  onlyOne( b, a );// true
  onlyOne( b, a, b, b, b );// true


```

**|| 和 &&**

&& 和 || 运算符的返回值并不一定是布尔类型，而是两个操作数其中一个的值。

```javascript
var a = 42; 
var b = "abc"; 
var c = null;
a || b; // 42
a && b;// "abc"
c || b; // "abc" 
c && b;// null
```

- || 和 && 首先会对第一个操作数(a 和 c)执行条件判断，如果其不是布尔值(如上例)就 先进行 ToBoolean 强制类型转换，然后再执行条件判断。
- 对于 || 来说，如果条件判断结果为 true 就返回第一个操作数(a 和 c)的值，如果为 false 就返回第二个操作数(b)的值。
- && 则相反，如果条件判断结果为 true 就返回第二个操作数(b)的值，如果为 false 就返回第一个操作数(a 和 c)的值。

```javascript
var a = 42; 
var b = null; 
var c = "foo";
if (a && (b || c)) { 
	console.log( "yep" );
}
```

a && (b || c)的结果实际上是"foo"而非true，然后再由if将foo强制类型转换为布尔值，所以最后结果为 true。

**宽松相等和严格相等**

- “== 允许在相等比较中进行强制类型转换，而 === 不允许。”
- 如果两个值的类型不同，就需要考虑有没有强制类型转换的必要，有就用 ==，没有就 强制类型转换用 ===，不用在乎性能。
- null  ==  undefined  //true

 **对象和非对象之间的相等比较**

(1) 如果 Type(x) 是字符串或数字，Type(y) 是对象，则返回 x == ToPrimitive(y) 的结果;

 (2) 如果 Type(x) 是对象，Type(y) 是字符串或数字，则返回 ToPromitive(x) == y 的结果。

```javascript
  var a = 42;
  var b = [ 42 ];
	a == b; // true
```

[ 42 ] 首先调用 ToPromitive 抽象操作，返回 "42"，变成 "42" == 42，然后又变成 42 == 42，最后二者相等。

```javascript
Number.prototype.valueOf = function() { return 3;};
new Number( 2 ) == 3;   // true
```

Number(2) 涉及 ToPrimitive 强制类型 转换，因此会调用 valueOf()。

**安全运用隐式强制类型转换**

- 如果两边的值中有 true 或者 false，千万不要使用 ==。
- 如果两边的值中有 []、"" 或者 0，尽量不要使用 ==。
- typeof x == "function"是100%安全的
- 要避免a < b中发生隐式强制类型转换，确保a和b为相同的类型
