# FrontEnd Interview

Try to leave out of Landray &amp; make more money.

## Questions

### 1.Base

### 2.JavaScript

#### 基本类型种类，有哪些

有两种数据类型，基本类型和引用类型。
基本类型：string, number, boolean, undefined, null, symbol(es6)
引用数据: object, array, function

---

#### typeof 是否能正确判断类型？instanceof 能正确判断对象的原理是什么？

对于基础类型来说，除了 null 都会正确显示类型
对于引用类型来说，除了函数都会显示 object
所以说 typeof 并**不能**准确判断变量到底是什么类型

---

#### let 为什么能解决 for 循环闭包问题

- 闭包产生

  内部函数依赖了外部作用域变量，即内部持有外部引用不释放（延续了引用变量的生命周期，延寿）
  变量的本质其实就是一个占位符，其值才是真正操作对象
  值可以是各语言的标量，也可以是内存地址（即通俗的引用类型）

- var VS let

  let 块级用域（ES5 之前的 js 不存在块级作用域）
  在一个块级作用域内，let 声明 的变量只会在该区域内存续
  var 不存在块的问题，可以在块外继续生活

- 与 for 的关系

  var 初始变量在 for 循环体内，是覆盖式的，用 C 的话来讲是共用体，即共用同一内存地址
  let 初始变量在每一次 for 循环中，都是一个独立的变量，拥有自己独立的内存地址。
  函数体内部引用了内部不存在的变量，会寻找上层作用域内的同名变量

- let + for + fn

  for 语句块内的函数引用了该层 let 声明变量
  函数引用了外部作用域 变量不会主动释放，即若该函数被调用该变量会存活于内存中。

- 小结

  for 循环体内定义 fn ，若函数体内用了 for 块 var 变量，在 for 语句外调用该函数，该函数采用的 var 值是循环结束后的 var 值
  而块内用 let 变量，与之同级的函数体用了该 let 变量，之后调用函数，函数使用的是定义时块内的 let 变量值。
  关键是否使用同一值（或址）

---

#### 手写一个 promise

简单够用版：

```js
function myPromise(constructor) {
  let self = this;
  self.status = 'pending'; //定义状态改变前的初始状态
  self.value = undefined; //定义状态为resolved的时候的状态
  self.reason = undefined; //定义状态为rejected的时候的状态
  function resolve(value) {
    //两个==="pending"，保证了状态的改变是不可逆的
    if (self.status === 'pending') {
      self.value = value;
      self.status = 'resolved';
    }
  }
  function reject(reason) {
    //两个==="pending"，保证了状态的改变是不可逆的
    if (self.status === 'pending') {
      self.reason = reason;
      self.status = 'rejected';
    }
  }
  //捕获构造异常
  try {
    constructor(resolve, reject);
  } catch (e) {
    reject(e);
  }
}
```

大厂专用版：

```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

function Promise(excutor) {
  let that = this; // 缓存当前promise实例对象
  that.status = PENDING; // 初始状态
  that.value = undefined; // fulfilled状态时 返回的信息
  that.reason = undefined; // rejected状态时 拒绝的原因
  that.onFulfilledCallbacks = []; // 存储fulfilled状态对应的onFulfilled函数
  that.onRejectedCallbacks = []; // 存储rejected状态对应的onRejected函数

  function resolve(value) {
    // value成功态时接收的终值
    if (value instanceof Promise) {
      return value.then(resolve, reject);
    }
    // 实践中要确保 onFulfilled 和 onRejected 方法异步执行，且应该在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。
    setTimeout(() => {
      // 调用resolve 回调对应onFulfilled函数
      if (that.status === PENDING) {
        // 只能由pending状态 => fulfilled状态 (避免调用多次resolve reject)
        that.status = FULFILLED;
        that.value = value;
        that.onFulfilledCallbacks.forEach((cb) => cb(that.value));
      }
    });
  }
  function reject(reason) {
    // reason失败态时接收的拒因
    setTimeout(() => {
      // 调用reject 回调对应onRejected函数
      if (that.status === PENDING) {
        // 只能由pending状态 => rejected状态 (避免调用多次resolve reject)
        that.status = REJECTED;
        that.reason = reason;
        that.onRejectedCallbacks.forEach((cb) => cb(that.reason));
      }
    });
  }

  // 捕获在excutor执行器中抛出的异常
  // new Promise((resolve, reject) => {
  //     throw new Error('error in excutor')
  // })
  try {
    excutor(resolve, reject);
  } catch (e) {
    reject(e);
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  const that = this;
  let newPromise;
  // 处理参数默认值 保证参数后续能够继续执行
  onFulfilled =
    typeof onFulfilled === 'function' ? onFulfilled : (value) => value;
  onRejected =
    typeof onRejected === 'function'
      ? onRejected
      : (reason) => {
          throw reason;
        };
  if (that.status === FULFILLED) {
    // 成功态
    return (newPromise = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onFulfilled(that.value);
          resolvePromise(newPromise, x, resolve, reject); // 新的promise resolve 上一个onFulfilled的返回值
        } catch (e) {
          reject(e); // 捕获前面onFulfilled中抛出的异常 then(onFulfilled, onRejected);
        }
      });
    }));
  }

  if (that.status === REJECTED) {
    // 失败态
    return (newPromise = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onRejected(that.reason);
          resolvePromise(newPromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
    }));
  }

  if (that.status === PENDING) {
    // 等待态
    // 当异步调用resolve/rejected时 将onFulfilled/onRejected收集暂存到集合中
    return (newPromise = new Promise((resolve, reject) => {
      that.onFulfilledCallbacks.push((value) => {
        try {
          let x = onFulfilled(value);
          resolvePromise(newPromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
      that.onRejectedCallbacks.push((reason) => {
        try {
          let x = onRejected(reason);
          resolvePromise(newPromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
    }));
  }
};
```

#### iframe 通信

1. postMessage
2. hash

#### Event Loop

宏任务 -> 所有入列微任务 -> 视图渲染 -
|
⬆️-----------------------------

---

#### 事件机制

分为捕获阶段、处于目标阶段、冒泡阶段三个阶段

---

### 3.CSS

### 4.React

#### React setState 是同步还是异步的，是否知晓其原理

- setState 只在合成事件和钩子函数中是“异步”的，在原生事件和 setTimeout 中都是同步的。
- setState 的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形成了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的 callback 拿到更新后的结果。
- setState 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和 setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次 setState，setState 的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时 setState 多个不同的值，在更新时会对其进行合并批量更新。

---

### 5.Git
