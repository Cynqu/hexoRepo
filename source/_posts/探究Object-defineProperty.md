---
title: 探究Object.defineProperty
date: 2021-04-26 18:10:31
tags: JavaScript
toc: true
excerpt: Object.defineProperty方法用于在对象上定义一个新属性，或者修改对象现有属性，并返回此对象。注意，请通过Object构造器调用此方法，而不是对象实例。
cover: https://images.unsplash.com/photo-1555949963-ff9fe0c870eb?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1350&q=80
categories: 
  - 前端
  - JavaScript
---
## 一、从零认识Object.defineProperty

### 基本用法及属性

Object.defineProperty方法用于在对象上定义一个新属性，或者修改对象现有属性，并返回此对象。注意，请通过Object构造器调用此方法，而不是对象实例。

方法基本语法如下：

```javascript
Object.defineProperty(obj, prop, descriptor)
```
添加属性与修改属性：
```javascript
// 添加属性
let o = {};
Object.defineProperty(o, 'name', {value:'echo'});
o.name;// 'echo'
// 修改现有属性
o.age = 27;
// 重返18岁
Object.defineProperty(o, 'age', {value:18});
o.age;// 18
```
语法中的obj是我们要 添加/修改 属性的对象，prop是我们希望 添加/修改 的属性名，而descriptor是我们添加/修改属性的具体描述，descriptor包含属性较多。
### descriptor中的数据描述符

对象里目前存在的属性描述符有两种主要形式：数据描述符和存取描述符。数据描述符是一个具有值的属性，该值可以是可写的，也可以是不可写的。存取描述符是由 getter 函数和 setter 函数所描述的属性。一个描述符只能是这两者其中之一，不能同时是两者。

![图片](https://uploader.shimo.im/f/791lGSdE7LEkVXEO.png!thumbnail?fileGuid=kktTWJHwXXvhCRXx)

descriptor中属性可分为了3类，数据描述符类（value，writable），存取描述符类（get，set），以及能与数据描述符或者存取描述符共存的共有属性（configurable，enumerable）。

#### writable

writable是一个布尔值，若不定义默认为false，表示此条属性只可读，不可修改

```javascript
let o = {};
Object.defineProperty(o, 'name', {
    value: '听风是风',
    writable: false
});
// 尝试修改name属性
o.name = '时间跳跃';
// 再次读取，结果并未修改成功
o.name;// 听风是风
```
#### const

const声明的变量是否可修改，准确来说可以改，分两种情况

```javascript
// 值为基本类型
const a = 1;
a = 2;// 报错
// 值为复杂类型
const b = [1];
b = [1,2];// 报错
const c = [1];
c[0] = 0;
c;// [0]
如果const声明变量赋值是基本类型，只要修改值一定报错；如果值是引用类型，比如值是一个数组，当我们直接使用赋值运算符整个替换数组还是会报错，但如果我们不是整个替换数组而是修改数组中某个元素可以发现并不会报错。

```
这是因为对于引用数据类型而言，变量保存的是数据存放的引用地址，比如b的例子，原本指向是[1]的地址，后面直接要把地址改成数组[1,2]的地址，这很明显是不允许的，所以报错了。但在c的例子中，我们只是把c地址指向的数组中的一位元素改变了，并未修改地址，这对于const是允许的。
这个特性对于writable也是适用的 :

```javascript
let o = {};
Object.defineProperty(o, 'age', {
    value: [27],
    writable: false
});
// 尝试修改name属性
o.age[0] = 18;
// 再次读取，修改成功
o.age; // 18
```
### descriptor中的存取描述符

存取描述符，也就是在vue中也出现过getter、setter方法

JavaScript中对象赋值与取值非常方便，有如下两种方式：

```javascript
let o = {};
// 通过.赋值取值
o.name = 'echo';
//通过[]赋值取值，这种常用于key为变量情况
o['age'] = 27;
```
存取描述符给了我们赋值/取值时**数据劫持**的机会，也就就是在赋值与取值时能自定义做一些操作
getter函数在获取属性值时触发，注意，是你为某个属性添加了getter在获取这个属性才会触发，如果未定义则为undefined，该函数的返回值将作为你访问的属性值。

setter函数在设置属性时触发，同理你得为这个属性提前定义这个方法才行，设置的值将作为参数传入到setter函数中，在这里我们可以加工数据，若未定义此方法默认也是undefined。

`getter`与`setter`模拟一个最常见的对象赋值与取值例子：

```javascript
let o = {};
o.name = '听风是风';
o.name; // '听风是风'
//使用get set模拟赋值取值操作
let age;
Object.defineProperty(o, 'age', {
    get() {
        // 直接返回age
        return age;
    },
    set(val) {
        // 赋值时触发，将值赋予变量age
        age = val;
    }
});
o.age = 18;
o.age; // 18
```
### descriptor中的共有属性

#### enumerable

enumerable值类型为Boolean，表示该属性是否可被枚举，Object.keys()用于获取对象可枚举属性

如果我们设置enumerable为false时：

```javascript
let o = {
    name: '听风是风'
};
Object.defineProperty(o, 'age', {
    value: 27,
    enumerable: false
});
// 无法获取keys
Object.keys(o); // ['name']
// 无法遍历访问
for (let i in o) {
    console.log(i); // 'name'
};


```
#### configurable

configurable的值也是Boolean，默认是false，configurable 特性表示对象的属性是否可以被删除，以及除 value 和 writable 特性外的其他特性是否可以被修改。

删除：

```javascript
let o = {
    name: '听风是风'
};
Object.defineProperty(o, 'age', {
    value: 27,
    configurable: false
});
delete o.name;//true
delete o.age;//false
o.name;//undefined
o.age;//18
```
其它属性的影响：
```javascript
var o = {};
Object.defineProperty(o, 'name', {
    get() {
        return '听风是风';
    },
    configurable: false
});
// 报错，尝试通过再配置修改name的configurable失败，因为已经定义过了configurable
Object.defineProperty(o, 'name', {
    configurable: true
});
//报错，尝试修改name的enumerable为true，失败，因为未定义默认为false
Object.defineProperty(o, 'name', {
    enumerable: true
});
//报错，尝试新增set函数，失败，一开始没定义set默认为undefined
Object.defineProperty(o, 'name', {
    set() {}
});
//尝试再定义get，报错，已经定义过了
Object.defineProperty(o, 'name', {
    get() {
        return 1;
    }
});
// 尝试添加数据描述符中的vaule，报错，数据描述符无法与存取描述符共存
Object.defineProperty(o, 'name', {
    value: 12
});
```
当configurable为false时，这些属性都无法被重新定义以及修改。
#### 其它注意点

1、对象属性描述符要么是数据描述符（value，writable），要么是存取描述符（get，set），不应该同时存在两者描述符。

```javascript
var o = {};
Object.defineProperty(o, 'name', {
    value: '时间跳跃',
    get() {
        return '听风是风';
    }
});
这个例子就会报错，其实不难理解，存取方法就是用来定义属性值的，value也是用来定义值的，同时定义程序也不知道该以哪个为准了，所以用了value/writable其一，就不能用get/set了；不过configurable与enumerable这两个属性可以与上面两种属性任意搭配。

```
各个属性是有默认值的，所以在用Object.defineProperty()时某个属性没定义不是代表没用这条属性，而是会用这条属性的默认值。

```javascript
let o = {};
Object.defineProperty(o, 'name', {
    value: '时间跳跃'
});
//等同于
Object.defineProperty(o, 'name', {
    value: '时间跳跃',
    writable: false,
    enumerable: false,
    configurable: false
});
```
```javascript
var o = {};
o.name = '听风是风';
//等同于
Object.defineProperty(o, 'name', {
    value: '听风是风',
    writable: true,
    enumerable: true,
    configurable: true
});
//等同于
let name = '听风是风';
Object.defineProperty(o, 'name', {
    get() {
        return name;
    },
    set(val) {
        name = val;
    },
    enumerable: true,
    configurable: true
});
```
2、属性分类与默认值
|configurable|enumerable|value|writable|get|set|    |
|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|**数据描述符**|**可以**|**可以**|**可以**|**可以**|**不可以**|**不可以**|
|**存取描述符**|**可以**|**可以**|**不可以**|**不可以**|**可以**|**可以**|
|**默认值**|**false**|**false**|**false**|**false**|**undefined**|**undefined**|



