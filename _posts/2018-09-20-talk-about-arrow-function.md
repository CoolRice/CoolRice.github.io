---
layout: post
title: 也谈箭头函数的 this 指向问题及相关
tags: [JavaScript, this]
bigimg: /img/2018-09-20-talk-about-arrow-function.jpg
---

注：本文和 [this（他喵的）到底是什么 — 理解 JavaScript 中的 this、call、apply 和 bind](https://coolrice.github.io/2018-09-17-this-keyword-call-apply-bind-javascript/)一起食用更佳。

最近翻译了一篇关于 this 的文章，很多人评价不错，但是原文没有讲到箭头函数的 `this`，所以我来补充一篇文章来专门讲解。

箭头函数是 ES6 添加的新语法，基础知识就不多介绍了，下面我们来讲一讲箭头函数的 `this` 怎么指向。


### 问题起源
在以往的函数中，`this` 有各种各样的指向(隐式绑定，显示绑定，new 绑定, window 绑定......)，虽然灵活方便，但由于不能在定义函数时而直到实际调用时才能知道 `this` 指向，很容易给开发者带来诸多困扰。

假如我们有下面这段代码（本文代码都是在浏览器下运行），

```
function User() {
  this.name = 'John';

  setTimeout(function greet() {
    console.log(`Hello, my name is ${this.name}`); // Hello, my name is
    console.log(this); // window
  }, 1000);
}

const user = new User();
```

`greet` 里的 `this` 可以由上一篇[文章](https://coolrice.github.io/2018-09-17-this-keyword-call-apply-bind-javascript/)的四个规则判断出来。对，因为没有显示绑定、隐式绑定或 `new` 绑定、所以直接得出结论 `this` 指向 `window`。但实际上我们想把 `this` 指向 `user` 对象！

以前是怎么解决的呢？看下面的代码：

**1. 使用闭包**

```
function User() {
  const self = this;
  this.name = 'John';

  setTimeout(function greet() {
    console.log(`Hello, my name is ${self.name}`); // Hello, my name is John
    console.log(self); // User {name: "John"}
  }, 1000);
}

const user = new User();
```
**2. 使用显示绑定 — `bind`**
```
function User() {
  this.name = 'John';

  setTimeout(function greet() {
    console.log(`Hello, my name is ${this.name}`); // Hello, my name is John
    console.log(this); // User {name: "John"}
  }.bind(this)(), 1000);
}

const user = new User();
```

**3. 利用 `setTimeout` 的可以传更多参数的特性**

其实第三种和第一种比较像，都用到了闭包。

```
function User() {
  this.name = 'John';

  setTimeout(function greet(self) {
    console.log(`Hello, my name is ${self.name}`); // Hello, my name is John
    console.log(self); // User {name: "John"}
  }, 1000, this);
}

const user = new User();
```
三种方法都可以解决问题，但是都要额外写冗余的代码来指定 `this`。

现在，**箭头函数**（Arrow Function）正是 ES6 引入来解决这个问题的，它可以轻松地让 `greet` 函数保持 `this` 指向 `user` 对象。

### 箭头函数如何解决

下面是箭头函数版本：

```
function User() {
  this.name = 'John';

  setTimeout(() => {
    console.log(`Hello, my name is ${this.name}`); // Hello, my name is John
    console.log(this); // User {name: "John"}
  }, 1000);
}

const user = new User();
```

完美，直接把普通函数改成箭头函数就能解决问题。

**箭头函数在自己的作用域内不绑定 `this`，即没有自己的 `this`，如果要使用 `this` ，就会指向定义时所在的作用域的 `this` 值**。在上面的代码中即指向 `User` 函数的 `this`，而 User 函数通过 `new` 绑定，所以 `this` 实际指向 `user` 对象。

如果上述代码在严格模式下运行会有影响吗？
```
function User() {
  this.name = 'John';

  setTimeout(() => {
    'use strict'
    console.log(`Hello, my name is ${this.name}`); // Hello, my name is John
    console.log(this); // User {name: "John"}
  }, 1000);
}

const user = new User();
```
答案是没有影响。因为箭头函数没有自己的 `this`，它的 `this` 来自于 `User` 的 `this`，**只要 `User` 的 `this` 不变，箭头函数的 `this` 也保持不变**。

那么使用 `bind`，`call` 或者 `apply` 呢？

```
function User() {
  this.name = 'John';

  setTimeout((() => {
    console.log(`Hello, my name is ${this.name}`); // Hello, my name is John
    console.log(this); // User {name: "John"}
  }).bind('no body'), 1000);
}

const user = new User();
```
答案还是没有影响。因为箭头函数没有自己的 `this`，使用 `bind`，`call` 或者 `apply` 时，箭头函数会自动忽略掉 `bind` 的第一个参数，即 `thisArg`。

总结：**箭头函数在自己的作用域内没有自己的 `this`，如果要使用 `this` ，就会指向定义时所在的作用域的 `this` 值**。

欢迎在下方评论交流哦！
