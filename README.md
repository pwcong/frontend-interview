# FrontEnd Interview
Try to leave out of Landray &amp; make more money.

## Questions

### 1.Base


### 2.JavaScript

#### 基本类型种类，有哪些
有两种数据类型，基本类型和引用类型。
基本类型：string, number, boolean, undefined, null, symbol(es6)
引用数据: object, array, function

#### typeof 是否能正确判断类型？instanceof 能正确判断对象的原理是什么？
```js
typeof 1 // 'number'
typeof '1' // 'string'
typeof undefined // 'undefined'
typeof true // 'boolean'
typeof Symbol() // 'symbol'
```

typeof 对于对象来说，除了函数都会显示 object，所以说 typeof 并**不能**准确判断变量到底是什么类型

```js
typeof [] // 'object'
typeof {} // 'object'
typeof console.log // 'function'
```

#### let为什么能解决for循环闭包问题

闭包产生
内部函数依赖了外部作用域变量，即内部持有外部引用不释放（延续了引用变量的生命周期，延寿）
变量的本质其实就是一个占位符，其值才是真正操作对象
值可以是各语言的标量，也可以是内存地址（即通俗的引用类型）

var VS let
let 块级用域（ES5之前的js不存在块级作用域）
在一个块级作用域内，let 声明 的变量只会在该区域内存续
var 不存在块的问题，可以在块外继续生活

与for的关系
var 初始变量在for循环体内，是覆盖式的，用C的话来讲是共用体，即共用同一内存地址
let初始变量在每一次for循环中，都是一个独立的变量，拥有自己独立的内存地址。
函数体内部引用了内部不存在的变量，会寻找上层作用域内的同名变量

let + for + fn
for语句块内的函数引用了该层let声明变量
函数引用了外部作用域 变量不会主动释放，即若该函数被调用该变量会存活于内存中。
小结

for循环体内定义fn ，若函数体内用了for块var变量，在for语句外调用该函数，该函数采用的var值是循环结束后的var值
而块内用let变量，与之同级的函数体用了该let变量，之后调用函数，函数使用的是定义时块内的let变量值。
关键是否使用同一值（或址）

### 3.CSS


### 4.React


### 5.Git