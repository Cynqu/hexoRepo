---
title: typescript进阶之重难点梳理
date: 2021-04-27 10:32:18
tags: typeScript
toc: true
excerpt: typescript进阶之重难点梳理
cover: https://www.typescriptlang.org/images/index/ts-conf-keynote.jpg
categories: 
  - 前端
  - typeScript
---
## 一、可索引类型

**「ts 的核心原则之一就是对值所具有的结构进行类型检查。」**它有时被称之为“鸭式辩型法”或“结构性子类型”。而接口就是其中的契约。可索引类型也是接口的一种表现形式。

![图片](https://uploader.shimo.im/f/NgEyvMNW4ri7hYVp.png!thumbnail?fileGuid=q6wYjVxXyGrJWdpT)

`StringArray`接口，它具有索引签名。 这个索引签名表示了当用`number`去索引`StringArray`时会得到`string`类型的返回值。

Typescript支持两种索引签名：字符串和数字。 可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。

这是因为当使用`number`来索引时，JavaScript会将它转换成`string`然后再去索引对象。 也就是说用100（一个number）去索引等同于使用"100"（一个string）去索引，因此两者需要保持一致。

![图片](https://uploader.shimo.im/f/cdNZf90VGOjBVZnQ.png!thumbnail?fileGuid=q6wYjVxXyGrJWdpT)

name的类型与字符串索引类型不匹配

![图片](https://uploader.shimo.im/f/JLpqRrZzeWcj52VG.png!thumbnail?fileGuid=q6wYjVxXyGrJWdpT)

也可以将索引签名设置为只读，这样就可以防止给索引赋值

![图片](https://uploader.shimo.im/f/kBgbNYFGywFqVVh0.png!thumbnail?fileGuid=q6wYjVxXyGrJWdpT)

## 二、interface  和 type 关键字

`interface`和`type`两个关键字的含义和功能都非常的接近。这里罗列下这两个主要的区别：

`interface`：

* 同名的`interface`自动聚合，也可以跟同名的`class`自动聚合
* 只能表示`object`、`class`、`function`类型

`type`:

* 不仅仅能够表示`object`、`class`、`function`
* 不能重名（自然不存在同名聚合了），扩展已有的`type`需要创建新`type`
* 支持复杂的类型操作
### 1、Objects/Functions

![图片](https://uploader.shimo.im/f/QLDFZY64Ws9L7hOz.png!thumbnail?fileGuid=q6wYjVxXyGrJWdpT)

![图片](https://uploader.shimo.im/f/MvjczONSJNdir6mv.png!thumbnail?fileGuid=q6wYjVxXyGrJWdpT)

### 2、其他数据类型

与`interface`不同，`type`还可以用来标书其他的类型，比如基本数据类型、元素、并集等

![图片](https://uploader.shimo.im/f/RnBnhswsrsPlxSVV.png!thumbnail?fileGuid=q6wYjVxXyGrJWdpT)

### 3、Extend

**「interface 和 type 彼此并不互斥」**。

#### interface extends interface

```typescript
interface PartialPointX {x:number;};
interface Point extends PartialPointX {y:number;};
```
#### type extends type

```typescript
type PartialPointX = {x:number;};
type Point = PartialPointX & {y:number;};
```
#### interface extends type

```typescript
type PartialPointX = {x:number;};
interface Point extends PartialPointX {y:number;};
```
#### type extends interface

```typescript
interface ParticalPointX = {x:number;};
type Point = ParticalPointX & {y:number};
```

### 4、implement

一个类，可以以完全相同的形式去实现`interface`或者`type`。但是，类和接口都被视为**静态蓝图（static blueprints）**，因此，他们不能实现/继承 联合类型的`type`

```typescript
interface Point {
  x: number;
  y: number;
}
class SomePoint implements Point {
  x: 1;
  y: 2;
}
type Point2 = {
  x: number;
  y: number;
};
class SomePoint2 implements Point2 {
  x: 1;
  y: 2;
}
type PartialPoint = { x: number; } | { y: number; };
// FIXME: can not implement a union type
class SomePartialPoint implements PartialPoint {
  x: 1;
  y: 2;
}
```

### 5、声明合并

和`type`不同，`interface`可以被重复定义，并且会被自动聚合

```typescript
interface Point {x:number;};
interface Point {y:number;};
const point:Pint = {x:1,y:2};
```

### 6、only interface can

在实际开发中，有的时候也会遇到`interface`能够表达，但是`type`做不到的情况：**「给函数挂载属性」**

```typescript
interface FuncWithAttachment {
  (param: string): boolean;
  someProperty: number;
}
const testFunc: FuncWithAttachment = function(param: string) {
  return param.indexOf("Neal") > -1;
};
const result = testFunc("Nealyang"); // 有类型提醒
testFunc.someProperty = 4;
```

## 三、泛型

泛型就是指定一个表示类型的变量，用它来代替某个实际的类型用于编程，而后再通过实际运行或推导的类型来对其进行替换，以达到一段使用泛型程序可以实际适应不同类型的目的。说白了，**「泛型就是不预先确定的数据类型，具体的类型在使用的时候再确定的一种类型约束规范」**。

泛型可以应用于`function`、`interface`、`type`或者`class`中。但是注意，**「泛型不能应用于类的静态成员」**

```typescript
function log<T>(value: T): T {
    console.log(value);
    return value;
}
// 两种调用方式
log<string[]>(['a', ',b', 'c'])
log(['a', ',b', 'c'])
log('Nealyang')
```
* 泛型类型、泛型接口
```typescript
type Log = <T>(value: T) => T
let myLog: Log = log
interface Log<T> {
    (value: T): T
}
let myLog: Log<number> = log // 泛型约束了整个接口，实现的时候必须指定类型。如果不指定类型，就在定义的之后指定一个默认的类型
myLog(1)
```
**「也可以把泛型变量理解为函数的参数，只不过是另一个维度的参数，是代表类型而不是代表值的参数。」**
```typescript
class Log<T> { // 泛型不能应用于类的静态成员
    run(value: T) {
        console.log(value)
        return value
    }
}
let log1 = new Log<number>() //实例化的时候可以显示的传入泛型的类型
log1.run(1)
let log2 = new Log()
log2.run({ a: 1 }) //也可以不传入类型参数，当不指定的时候，value 的值就可以是任意的值
```
类型约束，需预定义一个接口
```typescript
interface Length {
    length: number
}
function logAdvance<T extends Length>(value: T): T {
    console.log(value, value.length);
    return value;
}
// 输入的参数不管是什么类型，都必须具有 length 属性
logAdvance([1])
logAdvance('123')
logAdvance({ length: 3 })
```
泛型的好处：
* 函数和类可以轻松的支持多种类型，增强程序的扩展性
* 不必写多条函数重载，冗长的联合类型声明，增强代码的可读性
* 灵活控制类型之间的约束
## 四、类型断言

```typescript
const nealyang = {};
nealyang.enName = 'Nealyang'; // Error: 'enName' 属性不存在于 ‘{}’
nealyang.cnName = '一凨'; // Error: 'cnName' 属性不存在于 '{}'
interface INealyang = {
  enName:string;
  cnName：string;
}
const nealyang = {} as INealyang; // const nealyang = <INealyang>{};
nealyang.enName = 'Nealyang';
nealyang.cnName = '一凨'; 
```
类型断言比较简单，其实就是“纠正”`ts`对类型的判断。
需要注意一下两点即可：

* 推荐类型断言的预发使用`as`关键字，而不是`<>`,防止歧义
* 类型断言并非类型转换，类型断言发生在编译阶段。类型转换发生在运行时



## 五、keyof

**「keyof 是索引类型操作符」**。用于获取一个“常量”的类型,这里的“常量”是指任何可以在编译期确定的东西，例如`const`、`function`、`class`等。

它是从**「实际运行代码」**通向**「类型系统」**的单行道。理论上，任何运行时的符号名想要为类型系统所用，都要加上`typeof`。

在使用`class`时，`class`名表示实例类型，`typeof class`表示`class`本身类型。是的，这个关键字和 js 的`typeof`关键字重名了 。

假设 T 是一个类型，那么`keyof T`产生的类型就是`T`的属性名称字符串字面量类型构成的联合类型。

```typescript
interface IQZQD{
    cnName:string;
    age:number;
    author:string;
}
type ant = keyof IQZQD;
```
在`vscode`上，可以看到`ts`推断出来的`ant`：
注意，如果`T`是带有字符串索引的类型，那么`keyof T`是`string`或者`number`类型。

索引签名参数类型必须为 "string" 或 "number"

```typescript
interface Map<T> {
  [key: string]: T;
}
//T[U]是索引访问操作符;U是一个属性名称。
let keys: keyof Map<number>; //string | number
let value: Map<number>['antzone'];//number
```
## 六、extends

`extends`即为扩展、继承。在 ts 中，**「extends 关键字既可以来扩展已有的类型，也可以对类型进行条件限定」**。在扩展已有类型时，不可以进行类型冲突的覆盖操作。例如，基类型中键`a`为`string`，在扩展出的类型中无法将其改为`number`。

```typescript
type num = {
  num:number;
}
interface IStrNum extends num {
  str:string;
}
// 与上面等价
type TStrNum = A & {
  str:string;
}
```
在 ts 中，还可以通过条件类型进行一些三目操作：`T extends U ? X : Y`
```typescript
type IsEqualType<A , B> = A extends B ? (B extends A ? true : false) : false;
type NumberEqualsToString = IsEqualType<number,string>; // false
type NumberEqualsToNumber = IsEqualType<number,number>; // true
```

未完待续……

