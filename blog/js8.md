**1.事件模型**

```javascript
let button = document.getElementById("my-btn");
button.onclick = function(event) {
	console.log("Clicked");
};
```

 `console.log("Clicked") `直到 `button` 被点击后才会被执行。当 `button` 被点击，赋值给 `onclick` 的函数就被添加到作业队列的尾部，并在队列前部所有任务结束之后再执行。事件在响应用户交互或类似的低频功能时，事件很有用，但在面对更复杂的需求时仍然不够灵活。

**2.回调模式**

回调函数模式类 似于事件模型，因为异步代码也会在后面的一个时间点才执行。不同之处在于需要调用的函数是作为参数传入的。以node为例：

```javascript
readFile("example.txt", function(err, contents) {
  if (err) {
  	throw err;
  }
  	console.log(contents);
 });
console.log("Hi!");
```

`readFile() `会立即开始执行，并在开始读取磁盘时暂停。这意味着 `console.log("Hi!")` 会在 `readFile() `被调用后立即进行输出，要早于 `console.log(contents) `的打印操作。当` readFile() `结束操作后，它会将回调函数以及相关参数作为一个新的作业添加到作业队列的尾部。在之前的作业全部结束后，该作业才会执行。

**回调地狱**

```javascript
method1(function(err, result) {
  if (err) {
    throw err;
  }
  method2(function(err, result) {
    if (err) {
      throw err;
    }
      method3(function(err, result) {
      if (err) {
      throw err;
      }
        method4(function(err, result) {
          if (err) {
            throw err;
          }
          method5(result);
          });
      });
  });
});
```

当回调函数嵌套过多时，会陷入复杂的回调地狱里，不易于理解和调式。当需要追踪多个回调函数并做清理操作， Promise 能大幅度改善这种情况。

**3.Promise 模式**

`Promise` 的生命周期

每个 `Promise` 都会经历一个短暂的生命周期，初始为挂起态（ pending state），这表示异 操作尚未结束。一个挂起的` Promise` 也被认为是未决的（ unsettled ）。一旦异步操作结束，`Promise` 就会被认为是已决的（ settled ），并进入两种可能状态之一： 

1. 已完成（ fulfilled ）： `Promise` 的异步操作已成功结束；
2. 已拒绝（ rejected ）： `Promise` 的异步操作未成功结束，可能是一个错误，或由其他原 因导致。

内部的` [[PromiseState]]` 属性会被设置为 `"pending"` 、 `"fulfilled"` 或 `"rejected"` ，以反映 `Promise` 的状态。该属性并未在 `Promise` 对象上被暴露出来，因此你无法以编程方式判断 `Promise` 到底处于哪种状态。不过可以使用` then()` 方法在 `Promise` 的状态改变时执行一些特定操作。

**创建 unsettled 的 Promise**

> 使用 Promise 构造器来创建。此构造器接受单个参数：一个被称为执行器（ executor ）的函数，包含初始化 Promise 的代码。该执行器会被传递两个名为 resolve() 与 reject() 的函数作为参数。 resolve() 函数在执行器成功结束时被调用，用于示意该 Promise 已经准备好被决议（ resolved ），而 reject() 函数则表明执行器的操作已失败。

在 Node.js 中使用Promise ，实现readFile() 函数：

```javaScript
// Node.js 范例
let fs = require("fs");
function readFile(filename) {
  return new Promise(function(resolve, reject) {
    // 触发异步操作
    fs.readFile(filename, { encoding: "utf8" }, function(err, contents) {
      // 检查错误
      if (err) {
        reject(err);
          return;
        }
      // 读取成功
        resolve(contents);
      });
    });
}
let promise = readFile("example.txt");
// 同时监听完成与拒绝
promise.then(function(contents) {
  // 完成
  console.log(contents);
}, function(err) {
  // 拒绝
  console.error(err.message);
});
```

执行器会在 readFile() 被调用时立即运行。当 resolve() 或 reject() 在执行器内部被调用时，一个作业被添加到作业队列中，以便决议（ resolve ）这个 Promise 。这被称 为作业调度（ job scheduling ）。Promise 的执行器会立即执行，早于源代码中在其之后的任何 代码。例如：

```javaScript
let promise = new Promise(function(resolve, reject) {
  console.log("Promise");
  resolve();
});
console.log("Hi!");
//输出：Promise， Hi! 
```

当调用`then`后：

```javaScript
let promise = new Promise(function(resolve, reject) {
  console.log("Promise");
  resolve();
});
promise.then(function() {
	console.log("Resolved.");
});
console.log("Hi!");
// 输出：Promise，Hi！，Resolved
```

完成处理函数与拒绝处理函数总是会在执 行器的操作结束后被添加到作业队列的尾部。

**创建 settled 的 Promise**

1.使用 Promise.resolve()

`Promise.resolve()` 方法接受单个参数并会返回一个处于完成态的 `Promise` 

```javascript
let promise = Promise.resolve(42);
promise.then(function(value) {
	console.log(value); // 42
});
```

2.使用 Promise.reject()

 `Promise.reject()` 方法来创建一个已拒绝的 `Promise`

```javascript
let promise = Promise.reject(42);
promise.catch(function(value) {
	console.log(value); // 42
});
```

> 若传递一个 Promise 给 Promise.resolve() 或 Promise.reject() 方法，该 Promise 会不作修改原样返回。

传递非 Promise 的 Thenable

```javascript
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
let p1 = Promise.resolve(thenable);
p1.then(function(value) {
	console.log(value); // 42
});
```

`Promise.resolve()` 与` Promise.reject() `都能接受非 `Promise` 的 t`henable` 作为参数。当传 入了非 `Promise `的 `thenable` 时，这些方法会创建一个新的 `Promise` ，此 `Promise `会在` then()` 函数之后被调用。

**Promise链**

对 then() 或 catch() 的调用实际上创建并返回了另一个 Promise ，仅当前一个 Promise 被完成或拒绝时，后一个 Promise 才会被决议.

```javascript
let p1 = new Promise(function(resolve, reject) {
	resolve(42);
});
p1.then(function(value) {
	console.log(value);
}).then(function() {
	console.log("Finished");
});
//42
//Finished
```

在 Promise 链中返回值

```javascript
let p1 = new Promise(function(resolve, reject) {
	resolve(42);
});
p1.then(function(value) {
	console.log(value); // "42"
	return value + 1;
}).then(function(value) {
	console.log(value); // "43"
});
```

在 Promise 链中返回 Promise

```javascript
let p1 = new Promise(function(resolve, reject) {
	resolve(42);
});
let p2 = new Promise(function(resolve, reject) {
	resolve(43);
});
p1.then(function(value) {
  // 第一个完成处理函数
  console.log(value); // 42
  return p2;
}).then(function(value) {
  // 第二个完成处理函数
  console.log(value); // 43
});
```

**响应多个Promise**

ES6 提供了能监视多个 Promise 的两个方法： Promise.all() 与 Promise.race() 。

Promise.all() 方法接收单个可迭代对象（如数组）作为参数，并返回一个 Promise 。这个 可迭代对象的元素都是 Promise ，只有在它们都完成后，所返回的 Promise 才会被完成。例如：

```javaScript
let p1 = new Promise(function(resolve, reject) {
	resolve(42);
});
let p2 = new Promise(function(resolve, reject) {
	resolve(43);
});
let p3 = new Promise(function(resolve, reject) {
	resolve(44);
});
let p4 = Promise.all([p1, p2, p3]);
p4.then(function(value) {
  console.log(Array.isArray(value)); // true
  console.log(value[0]); // 42
  console.log(value[1]); // 43
  console.log(value[2]); // 44
});
```

Promise.race() 与等待所有 Promise 完成的 Promise.all() 方法不同，在来源 Promise 中任意一个被完成时， Promise.race() 方法所返回的 Promise 就能作出响应。例如：

```javaScript
let p1 = Promise.resolve(42);
let p2 = new Promise(function(resolve, reject) {
 resolve(43);
});
let p3 = new Promise(function(resolve, reject) {
 resolve(44);
});
let p4 = Promise.race([p1, p2, p3]);
p4.then(function(value) {
 console.log(value); // 42
});
```

