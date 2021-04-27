---
title: express学习笔记
date: 2021-04-27 15:00:17
tags: node
toc: true
excerpt: Express 支持多种不同的视图引擎，它们有不同层次的抽象。
cover: https://images.unsplash.com/photo-1550745165-9bc0b252726f?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80
categories: 
  - 前端
  - node
---
## 安装

```plain
$ mkdir myapp
$ cd myapp
$ npm init
$ npm install express --save
```
### 全局安装express-generator

```plain
npm install express-generator --save -g
```
## Hello world example

```plain
const express = require('express')
const app = express()
const port = 3000
// 定制 404 页面
app.use(function(req, res) {
  res.type("text/plain");
  res.status(404);
  res.send("404 - Not Found");
});
// 定制 500 页面
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.type("text/plain");
  res.status(500);
  res.send("500 - Server Error");
});
app.get('/', (req, res) => res.send('Hello World!'))
app.listen(port, () => console.log(`Example app listening on port ${port}!`))
// app.use 是 Express 添加中间件的一种方法
```
## 初始化项目

```plain
$ express --view=Jade myapp
```
Express 支持多种不同的视图引擎，它们有不同层次的抽象。

支持 (ejs|hbs|hjs|jade|pug|twig|vash) （默认是 jade 模板引擎）

## 请求报头

发送到服务器的并不只是 URL,会发送“用户代理”信息（浏览器、操作系统和硬件设备）和其他一些信息。所有能够确保你了解请求对象头文件属性的信息都将会作为请求报头发送。如果想查看浏览器发送的信息，可以创建一个非常简单的 Express 路由 来展示一下：

```plain
app.get("/headers", function(req, res) {
  res.set("Content-Type", "text/plain");
  var s = "";
  for (var name in req.headers) {
   s += name + ": " + req.headers[name] + "\n";
  }
  res.send(s);
});
```
## 响应报头

响应报头还经常会包含一些关于服务器的信息，一般会指出服务器的类型，有时甚至会包含操作系统的详细信息。返回服务器信息存在一个问题，那就是它会给黑客一个可乘之机，从而使站点陷入危险。非常重视安全的服务器经常忽略此信息，甚至提供虚假信息。禁用 Express 的 X-Powered-By 头信息

```plain
app.disable("x-powered-by");
```
### 请求体

除请求报头外，请求还有一个主体（就像作为实际内容返回的响应主体一样）

1. 一般 GET 请求没有主体内
2. POST 请求体最常见的媒体类型是 application/x-www-form-urlendcoded,是键值对集合的简单编码，用 & 分隔（基本上和查询字符串的格式一样）
3. POST 请求需要支持文件上传，则媒体类型是 multipart/form-data ，它是一种更为复杂的格式
4. AJAX 请求，它可以使用 application/json
### 请求对象

|方法|说明|
|:----|:----|
|req.params|一个数组，包含命名过的路由参数|
|req.param(name)|返回命名的路由参数，或者 GET 请求或 POST 请求参数|
|req.query|一个对象，包含以键值对存放的查询字符串参数（通常称为 GET 请求参数）|
|req.body|一个对象，包含 POST 请求参数,**需要中间件能够解析请求正文内容类型**|
|req.route|关于当前匹配路由的信息。主要用于路由调试|
|req.cookies/req.singnedCookies|一个对象，包含从客户端传递过来的 cookies 值|
|req.headers|从客户端接收到的请求报头。|
|req.accepts([types])|用来确定客户端是否接受一个或一组指定的类型（可选类型可以是单个的 MIME 类型，如 application/json 、一个逗号分隔集合或是一个数组）|
|req.ip|客户端的 IP 地址。|
|req.path|请求路径（不包含协议、主机、端口或查询字符串）|
|req.host|一个简便的方法，用来返回客户端所报告的主机名。这些信息可以伪造，所以不应该用于安全目的|
|req.xhr|一个简便属性，如果请求由 Ajax 发起将会返回 true|
|req.protocol|用于标识请求的协议（ http 或 https ）|
|req.secure|一个简便属性，如果连接是安全的，将返回 true 。等同于 req.protocol==='https'|
|req.url/req.originalUrl|这些属性返回了路径和查询字符串（它们不包含协议、主机或端口）|
|req.acceptedLanguages|用来返回客户端首选的一组（人类的）语言。这些信息是从请求报头中解析而来的|

### 响应对象

|方法|说明|
|:----|:----|:----|
|res.status(code)<br>|设置 HTTP 状态代码。Express 默认为 200（成功）   res.set(name,value) 设置响应头。这通常不需要手动设置|
|res.clearCookie(name,[options])|设置或清除客户端 cookies 值。需要中间件支持   res.redirect([status],url) 重定向浏览器。默认重定向代码是 302（建立）|
|res.send(body)|res.send(status,body) 向客户端发送响应及可选的状态码。Express 的默认内容类型是 text/html|
|res.json(json)|res.json(status,json) 向客户端发送 JSON 以及可选的状态码|

### 内容渲染

渲染内容用 res.render ，如果想写一个快速测试页，也许会用到 res.send

req.query 得到查询字符串的值，使用 req.session 得到会话值，或使用 req.cookie/req.singedCookies 得到 cookies 值

```plain
app.get("/about", function(req, res) {
  res.render("about");
});
```
#### 200以外响应代码

```plain
app.get("/error", function(req, res) {
  res.status(500);
  res.render("error");
});
//或者是一行代码
app.get("/error", function(req, res) {
  res.status(500).render("error");
});
```
#### 将上下文传递给视图，包括查询字符串

```plain
app.get("/greeting", function(req, res) {
  res.render("about", {
    message: "welcome",
    style: req.query.style,
    userid: req.cookie.userid,
    username: req.session.username
  });
});
```
#### 使用定制布局渲染视图

```plain
app.get("/custom-layout", function(req, res) {
  res.render("custom-layout", { layout: "custom" });
});
```
#### 添加错误处理程序

```plain
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).render("error");
});
```
#### 添加一个 404 处理程序

```plain
app.use(function(req, res) {
  res.status(404).render("not-found");
});
```
#### 基本表单处理

表单信息一般在 req.body 中（或者偶尔在 req.query 中）。你可以使用 req.xhr 来判断是 AJAX 请求还是浏览请求

```plain
// 必须引入中间件 body-parser
app.post("/process-contact", function(req, res) {
  console.log(
    "Received contact from " + req.body.name + " <" + req.body.email + ">"
  );
  // 保存到数据库……
  res.redirect(303, "/thank-you");
});
```
#### 路由路径和正则表达式

路由中指定的路径（比如 /foo）最终会被 Express 转换成一个正则表达式。某些正则表达式中的元字符可以用在路由路径中： + 、 ? 、 * 、 ( 和 ) 。我们看两个例子。比如你想用同一

个路由处理 /user 和 /username 两个 URL：

```plain
app.get("/user(name)?", function(req, res) {
res.render("user");
});
```


