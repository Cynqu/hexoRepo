---
title: typeScript学习笔记
date: 2021-04-27 10:27:37
tags: typeScript
toc: true
cover: https://images.unsplash.com/photo-1612198188060-c7c2a3b66eae?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80
excerpt: typeScript入门
categories: 
  - 前端
  - typeScript
---
# js 缺什么

* 类型检查
* 语言扩展
* vscode 补全

重塑类型思维。

强类型，弱类型语言。

java C# 等传统高级语言看 js：

* 面向对象，继承，多态，接口，命名空间，变量修饰，构造函数，访问器get set 静态属性
* 委托
* 泛型(随意的类型)
* 反射（看包内的数组，动态分析东西是什么）
* 集合 数据结构高级动态语言。
* 动态数组ArrayList Hashtable SortedList Stack Queue 匿名方法
* 拆箱
* 多线程 worker
# ts 入门

## 搭建 TS 运行环境

## 安装TypeScript

有两种主要的方式来获取TypeScript工具：

* 通过npm（Node.js包管理器）
* 安装Visual Studio的TypeScript插件

Visual Studio 2017和Visual Studio 2015 Update 3默认包含了TypeScript。 如果你的Visual Studio还没有安装TypeScript，你可以[下载](https://www.tslang.cn/#download-links?fileGuid=T8pDTPHX3jtDvkQP)它。

针对使用npm的用户：

```plain
> npm install -g typescript
```

## 基本类型

![图片](https://uploader.shimo.im/f/7yUt0ctNDtcKKcXS.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)

## 枚举

把一组数值 变为 有意义的名字。

用途：抽离常量，单独维护，节省记忆成本

![图片](https://uploader.shimo.im/f/7C0Pjug0Sbq0gneo.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)

## 接口 Interface

实现对对象的类型定义，不会编译，再开发阶段起辅助作用。

![图片](https://uploader.shimo.im/f/U2m7kSrliTk9ZxUU.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)

函数类型的接口

![图片](https://uploader.shimo.im/f/y24wQ2uWI1rIow3p.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)

## 类

也包含了 继承 抽象 接口 setter getter等。

类里的属性方法默认是 public。也可以被设置成 private protected

![图片](https://uploader.shimo.im/f/IngthkUvwNruyXfd.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)

## 类和接口的关系

![图片](https://uploader.shimo.im/f/jHGgCaANcyv4kHmo.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)

混合使用

![图片](https://uploader.shimo.im/f/ZTuB8xyYI9MRgGpY.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)

# 类型断言

会自动进行推断，从右到左。

上下文类型推断。

![图片](https://uploader.shimo.im/f/6c7T24jOdg2RbzzA.png!thumbnail?fileGuid=T8pDTPHX3jtDvkQP)





