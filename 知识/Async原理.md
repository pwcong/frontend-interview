# Async原理
## 1. generator函数

要解释async原理首先要理解generator函数怎么使用，下面是示例：

```js
function * gen() {
    yield 1
    yield 2
}

const t = gen()
console.log(t.next()) // {value: 1, done: false}
console.log(t.next()) // {value: 2, done: false}
console.log(t.next()) // {value: undefined, done: true}
```

上面代码可以看到generator函数执行到yield代码会暂停，执行next后会继续往下走，因此通过封装generator函数让其自执行即可实现async的效果。

## 2. co函数

```js
function co(gen) {
    return new Promise((resolve, reject) => {
        const fn = gen();
        function next(data) {
            let { value, done } = fn.next(data);
            if (done) return resolve(value);
            Promise.resolve(value).then((res) => {
                next(res);
            }, reject);
        }
        next();
      });
}
```

## 3. 测试

```js
function Asyncfn() {
    return co(function* () {
        const a = yield new Promise((resolve) =>
            setTimeout(() => resolve("1"), 1000)
        );
        console.log(a);
        const b = yield new Promise((resolve) =>
            setTimeout(() => resolve("2"), 1000)
        );
        console.log(b);
    }).then(() => {
        console.log("ok");
    });
}
```