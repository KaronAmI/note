# JS 总结之函数、作用域链

在 JavaScript 中，函数实际上是一个对象。

## 🏌 声明

JavaScript 用 function 关键字来声明一个函数：

```JavaScript
function fn () {

}
```

变体：函数表达式：

```JavaScript
var fn = function () {

}
```

这种没有函数名的函数被称为匿名函数表达式。

## 🤾‍ return

函数可以有返回值

```JavaScript
function fn () {
  return true
}
```

位于 return 之后的任何代码都不会执行：

```JavaScript
function fn () {
  return true
  console.log(false) // 永远不会执行
}
fn() // true
```

没有 return 或者只写 return，函数将返回 undefined：

```JavaScript
function fn () {
}
fn() // undefined
// 或者
function fn () {
  return
}
fn() // undefined
```

## ⛹ 参数

函数可以带有限个数或者不限个数的参数

```JavaScript
// 参数有限
function fn (a, b) {
  console.log(a, b)
}
// 参数不限
function fn (a, b, ..., argN) {
  console.log(a, b, ..., argN)
}
```

没有传值的命名参数，会被自动设置为 undefined

```JavaScript
// 参数有限
function fn (a, b) {
  console.log(b) // undefined
}
fn(1)
```

## 🚣 arguments

函数可以通过内部属性 arguments 这个**类数组**的对象来访问参数，**即便没有命名参数**：

```JavaScript
// 有命名参数
function fn (a, b) {
  console.log(arguments.length) // 2
}
fn(1, 2)

// 无命名参数
function fn () {
  console.log(arguments[0], arguments[1], arguments[2]) // 1, 2, 3
}
fn(1, 2, 3)
```

### ⛳️ 长度

arguments 的长度由传入的参数决定，并不是定义函数时决定的。

```JavaScript
function fn () {
  console.log(arguments.length) // 3
}
fn(1, 2, 3)
```

如果按定义函数是决定个的，那么此时的 arguments.length 应该为 0 而不为 3。

### 🏓 同步

arguments 对象中的值会自动反应到对应的命名参数，可以理解为同步，不过并不是因为它们读取了相同的内存空间，而**只是保持值同步而已**。

```JavaScript
function fn (a) {
  console.log(arguments[0]) // 1
  a = 2
  console.log(arguments[0]) // 2
  arguments[0] = 3
  console.log(a) // 3
}
fn(1)
```

严格模式下，重写 arguments 的值会导致错误。

### 🏸 callee

通过 callee 这个指针访问拥有这个 arguments 对象的函数

```JavaScript
function fn () {
  console.log(arguments.callee) // fn
}
fn()
```

### 🏒 类数组

长的跟数组一样，可以通过下标访问，如 arguments[0]，却**无法使用数组的内置方法**，如 forEach 等：

```JavaScript
function fn () {
  console.log(arguments[0], arguments[1]) // 1, 2
  console.log(arguments.forEach) // undefined
}
fn(1, 2)
```

通过对象那章知道，**可以用 call 或者 apply 借用函数**，所以 arguments 可以借用数组的内置方法：

```JavaScript
function fn () {
  Array.prototype.forEach.call(arguments, function (item) {
    console.log(item)
  })
}
fn(1, 2)
// 1
// 2
```

对于如此诡异的 arguments，我觉得还是少用为好。

## 🤺 this、 prototype

具体查看总结：

- [《关于 this 应该知道的几个点》](https://github.com/KaronAmI/blog/issues/21)
- [《原型》](https://github.com/KaronAmI/blog/issues/22)

## 🏋 按值传递

引用《JavaScript 高级程序设计》4.1.3 的一句话：

> ECMAScript 中所有函数的参数都是按值传递的，也就是说，把函数外部的值复制给函数内部的参数，就和把一个变量复制到另一个变量一样。

### 🎺 基本类型的参数传递

基本类型的传递很好理解，就是把变量复制给函数的参数，变量和参数是完全独立的两个个体：

```JavaScript
var name = 'jon'
function fn (a) {
  a = 'karon'
  console.log('a: ', a) // a: karon
}
fn(name)
console.log('name: ', name) // name: jon
```

用表格模拟过程：

<table>
  <tr>
    <td colspan=2>栈内存</td>
    <td>堆内存</td>
  </tr>
  <tr style="background: #f5f5f5">
    <td>name, a</td>
    <td>jon</td>
    <td></td>
  </tr>
</table>

将 a 复制为其他值后：

<table>
  <tr>
    <td colspan=2>栈内存</td>
    <td>堆内存</td>
  </tr>
  <tr style="background: #f5f5f5">
    <td>name</td>
    <td>jon</td>
    <td></td>
  </tr>
  <tr style="background: #f5f5f5">
    <td>a</td>
    <td>karon</td>
    <td></td>
  </tr>
</table>

### 🎻 引用类型的参数传递

```JavaScript
var obj = {
  name: 'jon'
}
function fn (a) {
  a.name = 'karon'
  console.log('a: ', a) // a: { name: 'karon' }
}
fn(obj)
console.log(obj) // name: { name: 'karon' }
```

嗯？说好的按值传递呢？我们尝试把 a 赋值为其他值，看看会不会改变了 obj 的值：

```JavaScript
var obj = {
  name: 'jon'
}
function fn (a) {
  a = 'karon'
  console.log('a: ', a) // a: karon
}
fn(obj)
console.log(obj) // name: { name: 'jon' }
```

### 🎸 真相浮出水面

**参数 a 只是复制了 obj 的引用**，所以 a 能找到对象 obj，自然能对其进行操作。一旦 a 赋值为其他属性了，obj 也不会改变什么。

用表格模拟过程：

<table>
  <tbody>
    <tr>
      <td colspan=2>栈内存</td>
      <td>堆内存</td>
    </tr>
    <tr style="background: #f5f5f5">
      <td>obj, a</td>
      <td>引用值</td>
      <td>{ name: 'jon' }</td>
    </tr>
  </tbody>
</table>

参数 a 只是 复制了 obj 的引用，所以 a 能找到存在堆内存中的对象，所以 a 能对堆内存中的对象进行修改后：

<table>
  <tbody>
    <tr>
      <td colspan=2>栈内存</td>
      <td>堆内存</td>
    </tr>
    <tr style="background: #f5f5f5">
      <td>obj, a</td>
      <td>引用值</td>
      <td>{ name: 'karon' }</td>
    </tr>
  </tbody>
</table>

将 a 复制为其他值后：

<table>
  <tbody>
    <tr>
      <td colspan=2>栈内存</td>
      <td>堆内存</td>
    </tr>
    <tr style="background: #f5f5f5">
      <td>obj</td>
      <td>引用值</td>
      <td>{ name: 'karon' }</td>
    </tr>
    <tr style="background: #f5f5f5">
      <td>a</td>
      <td>'karon'</td>
      <td></td>
    </tr>
  </tbody>
</table>

因此，**基本类型和引用类型的参数传递也是按值传递的**

## 🚴 作用域链

理解作用域链之前，我们需要理解**执行环境** 和 **变量对象**。

### 🍗 执行环境

执行环境定义了变量或者函数有权访问的其它数据，可以把执行环境理解为一个大管家。

执行环境分为**全局执行环境**和**函数执行环境**，全局执行环境被认为是 window 对象。而函数的执行环境则是由函数创建的。

每当一个函数被执行，就会被推入一个环境栈中，执行完就会被推出，环境栈最底下一直是全局执行环境，只有当关闭网页或者推出浏览器，全局执行环境才会被摧毁。

### 🍖 变量对象

每个执行环境都有一个变量对象，存放着环境中定义的所有变量和函数，是作用域链形成的前置条件。但我们无法直接使用这个变量对象，该对象主要是给 JS 引擎使用的。具体可以查看[《JS 总结之变量对象》](https://github.com/KaronAmI/blog/issues/27)。

### 🍤 作用域链的作用

而**作用域链**属于执行环境的一个变量，作用域链收集着所有有序的变量对象，函数执行环境中函数自身的变量对象（此时称为活动对象）放置在作用域链的最前端，如：

```
scope: [函数自身的变量对象，变量对象1，变量对象2，..., 全局执行环境的变量对象]
```

**作用域链保证了对执行环境有权访问的所有变量和函数的有序访问**。

```JavaScript
var a = 1
function fn1 () {
  var b = 2
  console.log(a，b) // 1, 2
  function fn2 () {
    var c = 3
    console.log(a, b, c) // 1, 2, 3
  }
  fn2()
}
fn1()
```

对于 fn2 来说，作用域链为： **fn2 执行环境**、**fn1 执行环境** 和 **全局执行环境** 的变量对象（所有变量和函数）。

对于 fn1 来说，作用域链为： **fn1 执行环境** 和 **全局执行环境** 的变量对象（所有变量和函数）。

总结为一句：函数内部能访问到函数外部的值，函数外部无法范围到函数内部的值。引出了闭包的概念，查看总结：[《JS 总结之闭包》](https://github.com/KaronAmI/blog/issues/26)

## 🏇 箭头函数

ES6 新语法，使用 => 定义一个函数：

```JavaScript
let fn = () => {}
```

当只有一个参数的时候，可以省略括号：

```JavaScript
let fn = a => {}
```

当只有一个返回值没有其他语句时，可以省略大括号：

```JavaScript
let fn = a => a

// 等同于
let fn = function (a) {
  return a
}
```

返回对象并且没有其他语句的时候，大括号需要括号包裹起来，**因为 js 引擎认为大括号是代码块**：

```JavaScript
let fn = a => ({ name: a })

// 等同于
let fn = function (a) {
  return { name: a }
}
```

箭头函数的特点：

1. **没有 this**，函数体内的 this 是定义时外部的 this
2. **不能被 new**，因为没有 this
3. **不可以使用 arguments**，可以使用 rest 代替
4. **不可以使用 yield 命令**，因此箭头函数不能用作 Generator 函数。

## 🚀 参考

- [《JavaScript 深入之参数按值传递》](https://github.com/mqyqingfeng/Blog/issues/10) by 冴羽
- [《ECMAScript 6 入门》函数的扩展 - 箭头函数](http://es6.ruanyifeng.com/#docs/function#%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0) by 阮一峰
- 《JavaScript 高级程序设计》3 基本概念、4.1.3 传递参数、4.2 执行环境及作用域
