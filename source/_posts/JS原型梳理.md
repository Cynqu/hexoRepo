---
title: JS原型梳理
date: 2021-04-26 17:47:55
tags: JavaScript
toc: true
excerpt: JS原型梳理
cover: https://images.unsplash.com/photo-1457369804613-52c61a468e7d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1350&q=80
categories: 
  - 前端
  - JavaScript
---
## 一、原型链图解

![图片](https://uploader.shimo.im/f/AsQiGDLcOFEEfrA0.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

* `function Foo`就是一个方法，比如JavaScript 中内置的`Array`、`String`等
* `function Object`就是一个`Object`
* `function Function`就是`Function`
* 以上都是`function`，所以`__proto__`都是`Function.prototype`
* `String、Array、Number、Function、Object`都是`function`
## 二、函数对象和普通对象

![图片](https://uploader.shimo.im/f/3U2PCFROs5cq72Hv.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

在 JavaScript 中，我们将对象分为函数对象和普通对象。**所谓的函数对象，其实就是 JavaScript 的用函数来模拟的类实现**。JavaScript 中的 Object 和 Function 就是典型的函数对象。

![图片](https://uploader.shimo.im/f/6jtCsme5p2djLlfm.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

不知道大家看到上述代码有没有一些疑惑的地方~别着急，我们一点一点梳理。

上述代码中，`obj1`，`obj2`，`obj3`，`obj4`都是普通对象，`fun1`，`fun2`，`fun3`都是`Function`的实例，也就是函数对象。

所以可以看出，**所有 Function 的实例都是函数对象，其他的均为普通对象，其中包括 Function 实例的实例**。

![图片](https://uploader.shimo.im/f/K9px83QMgry8cC9D.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

**JavaScript 中万物皆对象，而对象皆出自构造（构造函数）**。

![图片](https://uploader.shimo.im/f/cs5ljncSPxxxqoYC.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

## 三、__proto__

首先需要明确两点：`__proto__`和`constructor`是**对象**独有的。`prototype`属性是**函数**独有的；

但是在 JavaScript 中，函数也是对象，所以函数也拥有`__proto__`和`constructor`属性。

![图片](https://uploader.shimo.im/f/jmI2126QynwmaVho.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

ECMAScript 规范描述`prototype`是一个隐式引用，但之前的一些浏览器，已经私自实现了`__proto__`这个属性，使得可以通过`obj.__proto__`这个显式的属性访问，访问到被定义为隐式属性的`prototype`。

因此，情况是这样的，ECMAScript 规范说`prototype`应当是一个隐式引用:

* 通过`Object.getPrototypeOf(obj)`间接访问指定对象的`prototype`对象
* 通过`Object.setPrototypeOf(obj, anotherObj)`间接设置指定对象的`prototype`对象
* 部分浏览器提前开了`__proto__`的口子，使得可以通过`obj.__proto__`直接访问原型，通过`obj.__proto__ = anotherObj`直接设置原型
* ECMAScript 2015 规范只好向事实低头，将`__proto__`属性纳入了规范的一部分

从浏览器的打印结果我们可以看出，上图对象`a`存在一个`__proto__`属性。而事实上，他只是开发者工具方便开发者查看原型的故意渲染出来的一个虚拟节点。虽然我们可以查看，但实则并不存在该对象上。

`__proto__`属性既不能被`for in`遍历出来，也不能被`Object.keys(obj)`查找出来。

访问对象的`obj.__proto__`属性，默认走的是`Object.prototype`对象上`__proto__`属性的 get/set 方法。

![图片](https://uploader.shimo.im/f/hee0iRNlHihst7Ds.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

![图片](https://uploader.shimo.im/f/sHoWZhIS1fsrC9Cm.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

关于更多`__proto__`更深入的介绍，可以参看工业聚大佬的《深入理解 JavaScript 原型》一文。

这里我们需要知道的是，`__proto__`是对象所独有的，并且`__proto__`是**一个对象指向另一个对象**，也就是他的原型对象。我们也可以理解为父类对象。它的作用就是当你在访问一个对象属性的时候，如果该对象内部不存在这个属性，那么就回去它的`__proto__`属性所指向的对象（父类对象）上查找，如果父类对象依旧不存在这个属性，那么就回去其父类的`__proto__`属性所指向的父类的父类上去查找。以此类推，知道找到`null`。而这个查找的过程，也就构成了我们常说的**原型链**。

## 四、prototype

在规范里，prototype 被定义为：给其它对象提供共享属性的对象。`prototype`自己也是对象，只是被用以承担某个职能罢了.

所有对象，都可以作为另一个对象的`prototype`来用。

![图片](https://uploader.shimo.im/f/ygGMx7AatUkBW1d1.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

修改`__proto__`的关系图，我们添加了`prototype`,`prototype`是函数所独有的。**它的作用就是包含可以给特定类型的所有实例提供共享的属性和方法。它的含义就是函数的远行对象，**也就是这个函数所创建的实例的远行对象，正如上图：`nealyang.__proto__ === Person.prototype`。任何函数在创建的时候，都会默认给该函数添加`prototype`属性.

## 五、constructor

`constructor`属性也是对象所独有的，它是**一个对象指向一个函数，这个函数就是该对象的构造函数**。

注意，每一个对象都有其对应的构造函数，本身或者继承而来。单从`constructor`这个属性来讲，只有`prototype`对象才有。每个函数在创建的时候，JavaScript 会同时创建一个该函数对应的`prototype`对象，而`函数创建的对象.__proto__ === 该函数.prototype`，该`函数.prototype.constructor===该函数本身`，故通过函数创建的对象即使自己没有`constructor`属性，它也能通过`__proto__`找到对应的`constructor`，所以任何对象最终都可以找到其对应的构造函数。

唯一特殊的可能就是我开篇抛出来的一个问题。JavaScript 原型的老祖宗：`Function`。它是它自己的构造函数。所以`Function.prototype === Function.__proto`。

为了直观了解，我们在上面的图中，继续添加上`constructor`：

![图片](https://uploader.shimo.im/f/YbLjOJhhgsD0QFTS.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)

其中`constructor`属性，**虚线表示继承而来的 constructor 属性**。

`__proto__`介绍的原型链，我们在图中直观的标出来的话就是如下这个样子

![图片](https://uploader.shimo.im/f/4mp2oj5Nc5g2TjrE.png!thumbnail?fileGuid=QggPTGgHqtqXXtgr)


