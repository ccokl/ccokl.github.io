#### 创建一个对象的语法

```javascript
var myObj = { key: value
// ... };
```

```javascript
var myObj = new Object(); 
myObj.key = value;
```

#### js中的基础类型

`string,number,boolean,null,undefined,object`

#### js中的内置对象

`String,Number,Boolean,Object,Function,Array,Date,RegExp,Error`

在使用`string` 等创建的字面量形式时，js会隐试转换成其对应的内置对象，`null` 和 `undefined `没有对应的构造形式,`Date` 只有构造，没有字面量形式

#### 对象的中的属性

```javascript
var myObject = { 
	a: 2
};
myObject.a; // 2 
myObject["a"]; // 2
```

. 操作符要求属性名满足标识符的命名规范，而 [".."] 语法 可以接受任意 UTF-8/Unicode 字符串作为属性名。在对象中，属性名永远都是字符串。如果你使用 string(字面量)以外的其他值作为属性 名，那它首先会被转换为一个字符串。即使是数字也不例外

#### 可计算属性名

ES6 增加了可计算属性名`[prefix + "bar"]`

```javascript
var prefix = "foo";
var myObject = {
  [prefix + "bar"]:"hello", 
  [prefix + "baz"]: "world"
};
myObject["foobar"]; // hello 
myObject["foobaz"]; // world
```

#### 对象复制

``` javaScript
function anotherFunction() { /*..*/ }
var anotherObject = { 
  c: true
};
var anotherArray = [];
var myObject = { 
  a: 2,
  b: anotherObject, // 引用 
  c: anotherArray, // 引用
  d: anotherFunction
};
anotherArray.push( anotherObject, myObject );
```

以上代码myObject中的b，c只是引用了anotherObject，anotherArray，相当于b指向anotherObject，anotherObject指向c，而b并没有直接指向c，只是做了一层浅拷贝。

可以通过JSON实现一个深拷贝，`JSON.stringify( someObj )` 将对象someObj生成一个JSON字符串，但这种方法只适用于对象内的属性可以被JSON序列化的情况

```javascript
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

ES6 `Object.assign(..) ` 可以实现对象的浅拷贝

#### 对象的属性描述符

```javascript
var myObject = { a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" ); 
// {
// value: 2,
// writable: true, 可写
// enumerable: true,可枚举
// configurable: true 可配置
// }
```

`Object.getOwnPropertyDescriptor`获取对象属性的描述，

```javascript
var myObject = {};
Object.defineProperty( myObject, "a", { 
    value: 2,
    writable: true, 
    configurable: true, 
    enumerable: true
} );
myObject.a; // 2
```

`Object.defineProperty`为对象的属性定义描述

1. `Writable`决定是否可以修改属性的值。
2. `Configurable`只要属性是可配置的，就可以使用 defineProperty(..) 方法来修改属性描述符
3. `Enumerable`控制属性是否会出现在对象的属性枚举中

```javaScript 
var myObject = {};
Object.defineProperty( myObject, "FAVORITE_NUMBER", 
{ value: 42,
writable: false,
configurable: false } );

```

结合 `writable:false` 和 `configurable:false `就可以创建一个常量属性(不可修改、 重定义或者删除)

```javascript
var myObject = { 
  a:2
};
Object.preventExtensions( myObject );
myObject.b = 3; myObject.b; // undefined
```

`Object.preventExtensions( myObject );`禁止一个对象`myObject`添加新属性并且保留已有属性

```
Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用 Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。
```

```
Object.freeze(..) 会创建一个冻结对象，这个方法实际上会在一个现有对象上调用 Object.seal(..) 并把所有“数据访问”属性标记为 writable:false，这样就无法修改它们的值。
```

#### [[Get]]

```
var myObject = { a: 2
};
myObject.a; // 2
```

在语言规范中，myObject.a 在 myObject 上实际上是实现了 [[Get]] 操作,对象默认的内置 [[Get]] 操作首先在对象中查找是否有名称相同的属性， 如果找到就会返回这个属性的值,反之遍历可能存在的 [[Prototype]] 链上是否有属性a，没有[[Get]] 操作会返回值 undefined。

> 注意，这种方法和访问变量时是不一样的。如果你引用了一个当前词法作用域中不存在的变量，并不会像对象属性一样返回 undefined，而是会抛出一个 ReferenceError 异常

```javascript
var myObject = { 
a: undefined
};
myObject.a; // undefined 
myObject.b; // undefined
```

仅根据返回值无法判断出到底变量的值为 undefined 还是变量不存在

#### [[Put]]

[[Put]] 被触发时，如果已经存在这个属性，[[Put]] 算法大致会检查下面这些内容。

1. 属性是否是`访问描述符`,如果是并且存在setter就调用setter。
2. 属性的数据描述符中writable是否是false?如果是，在非严格模式下静默失败，在严格模式下抛出 TypeError 异常。
3. 如果都不是，将该值设置为属性的值。

> 对象默认的 [[Put]] 和 [[Get]] 操作分别可以控制属性值的设置和获取。

#### Getter和Setter“访问描述符”

在 ES5 中可以使用 getter 和 setter 部分改写默认操作，但是只能应用在单个属性上，无法应用在整个对象上。getter 是一个隐藏函数，会在获取属性值时调用。setter 也是一个隐藏函数,会在设置属性值时调用。对于访问描述符来说，JavaScript 会忽略它们的 value 和 writable 特性，取而代之的是关心 set 和 get(还有 configurable 和 enumerable)特性。

```javascript
var myObject = {
  // 给 a 定义一个 getter 
  get a() {
    return 2; 
}};
Object.defineProperty( myObject, "b",{
  // 描述符
  // 给 b 设置一个 getter
  get: function(){ return this.a * 2 },
  // 确保 b 会出现在对象的属性列表中
  enumerable: true
});
myObject.a; // 2 
myObject.b; // 4
```

不管是对象文字语法中的get a() { .. }，还是defineProperty(..)中的显式定义，二者 都会在对象中创建一个不包含值的属性，对于这个属性的访问会自动调用一个隐藏函数， 它的返回值会被当作属性访问的返回值

```javascript
var myObject = {
// 给 a 定义一个 getter 
  get a() {
		return this._a_; 
  },
// 给 a 定义一个 setter 
  set a(val) {
		this._a_ = val * 2; 
  }
};
myObject.a = 2; 
myObject.a; // 4
```

setter 会覆盖单个属性默认的 [[Put]](也被称为赋值)操作。通常来说 getter 和 setter 是成对出现的

#### 如何区分属性是否存在

```javascript
var myObject = { 
	a:2
};
("a" in myObject); // true 
("b" in myObject); // false
myObject.hasOwnProperty( "a" ); // true 
myObject.hasOwnProperty( "b" ); // false
```

in 操作符会检查属性是否在对象及其 [[Prototype]] 原型链中，hasOwnProperty(..) 只会检查属性是否在 myObject 对象中，不会检查 [[Prototype]] 链。

> 有 的 对 象 可 能 没 有 连 接 到 Object.prototype( 通 过 Object. create(null) 来创建)Object.prototype.hasOwnProperty. call(myObject,"a")

#### 枚举

```javascript
var myObject = { };
Object.defineProperty( myObject,
  "a",
  // 让 a 像普通属性一样可以枚举 
  { enumerable: true, value: 2 }
);
Object.defineProperty( myObject,
"b",
// 让b不可枚举
{ enumerable: false, value: 3 }
);
myObject.b; // 3
("b" in myObject); // true 
myObject.hasOwnProperty( "b" ); // true
// .......
for (var k in myObject) { 
  console.log( k, myObject[k] );
}
myObject.propertyIsEnumerable( "a" ); // true 
myObject.propertyIsEnumerable( "b" ); // false
Object.keys( myObject ); // ["a"] 
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
// "a" 2
```

propertyIsEnumerable(..) 会检查给定的属性名是否直接存在于对象中(而不是在原型链 上)并且满足 enumerable:true。

> for ... in...用在对象上，用在数组上不仅会包含所有数值索引，还会包含所有可枚举属性，数组用for循环
