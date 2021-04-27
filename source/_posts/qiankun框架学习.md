---
title: qiankun
date: 2021-04-26 15:37:52
tags: 框架
toc: true
excerpt: qiankun是基于single-spa 一般来说，微前端需要解决的问题分为两大类： 1. 应用的加载与切换 2. 应用的隔离与通信
cover: https://images.unsplash.com/photo-1550836445-05aac225729c?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1226&q=80
categories: 
  - 前端
  - 框架
---
# 一、qiankun与single-spa实现原理

qiankun是基于single-spa

一般来说，微前端需要解决的问题分为两大类：

1. 应用的加载与切换
2. 应用的隔离与通信

应用的加载与切换需要解决的问题包括：**路由问题、应用入口、应用加载**；

应用的隔离与通信需要解决的问题包括：**js隔离、css样式隔离、应用间通信**。

**single-spa**很好地解决了**路由**和**应用入口**两个问题，但并没有解决**应用加载**问题，而是将该问题暴露出来由使用者实现（一般可以用**system.js**或**原生script标签**来实现）；**qiankun**在此基础上封装了一个应用加载方案（即**import-html-entry**），并给出了js隔离、css样式隔离和应用间通信三个问题的解决方案，同时提供了预加载功能。

借助**single-spa**提供的能力，我们只能把不同的应用加载到一个页面内，但是很难保证这些应用不会互相干扰。而**qiankun**解决了这些问题，使得它成为一个更加完整的微前端运行时容器。

## 1、single-spa实现原理

### （1）基础概念

* **加载器**：也就是微前端架构的核心，主要用来调度子应用，决定何时展示哪个子应用， 可以把它理解成电源。
* **包装器**：有了加载器，可以把现有的应用包装，使得加载器可以使用它们，它相当于电源适配器。
* **主应用**：一般是包含所有子应用公共部分的项目—— 它相当于电器底座
* **子应用**：众多展示在主应用内容区的应用—— 它相当于你要使用的电器，所以是这么个概念：电源(加载器)→电源适配器(包装器)→️电器底座(主应用)→️电器(子应用)️

总的来说是这样一个流程：用户访问index.html后，浏览器运行加载器的js文件，加载器去配置文件，然后注册配置文件中配置的各个子应用后，首先加载主应用(菜单等)，再通过路由判定，动态远程加载子应用。

**SystemJS**一个运行于浏览器端的模块加载器，提供通用的模块导入途径，支持传统模块和ES6的模块。

**SystemJs**有两个版本，6.x版本是在浏览器中使用的，0.21版本的是在浏览器和node环境中使用的，两者的使用方式不同。(参考：[https://github.com/systemjs/systemjs](https://github.com/systemjs/systemjs?fileGuid=cYYhDWjhK9qcKW6w))

在微服务中相当于**加载器**的角色。

**singleSpa**

single-spa是一个在前端应用程序中将多个javascript应用集合在一起的框架。相当于**包装器**的角色

### （2）路由问题

single-spa是通过监听**hashChange**和**popState**这两个原生事件来检测路由变化的，它会根据路由的变化来加载对应的应用。

相关的代码可以在**single-spa**的**src/navigation/navigation-events.js**中找到：

```javascript
...
// 139行
if (isInBrowser) {
  // We will trigger an app change for any routing events.
  window.addEventListener("hashchange", urlReroute);
  window.addEventListener("popstate", urlReroute);
...
// 174行，劫持pushState和replaceState
  window.history.pushState = patchedUpdateState(
    window.history.pushState,
    "pushState"
  );
  window.history.replaceState = patchedUpdateState(
    window.history.replaceState,
    "replaceState"
  );
```
**single-spa**在检测到发生**hashChange**或**popState**事件时，会执行**urlReroute**函数，这里封装了它对路由问题的解决方案。
另外，它还劫持了原生的**pushState**和**replaceState**事件

urlReroute：

```javascript
function urlReroute() {
  reroute([], arguments);
} 
```
**reroute**函数就是**single-spa**解决路由问题的核心逻辑
```javascript
export function reroute(pendingPromises = [], eventArguments) {
  ...
  // getAppChanges会根据路由改变应用的状态，状态包含4类
  // 待清除、待卸载、待加载、待挂载
  const {
    appsToUnload,
    appsToUnmount,
    appsToLoad,
    appsToMount,
  } = getAppChanges();
  ...
  // 如果应用已启动，则调用performAppChanges加载和挂载应用
  // 否则，只加载未加载的应用
  if (isStarted()) {
    appChangeUnderway = true;
    appsThatChanged = appsToUnload.concat(
      appsToLoad,
      appsToUnmount,
      appsToMount
    );
    return performAppChanges();
  } else {
    appsThatChanged = appsToLoad;
    return loadApps();
  }
  ...
  function performAppChanges() {
    return Promise.resolve().then(() => {
      // 1. 派发应用更新前的自定义事件
      // 2. 执行应用暴露出的生命周期函数
      // appsToUnload -> unload生命周期钩子
      // appsToLoad -> 执行加载方法
      // appsToUnmount -> 卸载应用，并执行对应生命周期钩子
      // appsToMount -> 尝试引导和挂载应用
    })
  }
  ...
}
```
**single-spa**解决路由问题的主要逻辑
* 根据传入的参数**activeWhen**判断哪个应用需要加载，哪个应用需要卸载或清除，并将其push到对应的数组
* 如果应用已经启动，则进行应用加载或切换。针对应用的不同状态，直接执行应用自身暴露出的生命周期钩子函数即可。
* 如果应用未启动，则只去下载**appsToLoad**中的应用。

总的来看，当路由发生变化时，**hashChange**或**popState**会触发，这时**single-spa**会监听到，并触发**urlReroute**；接着它会调用**reroute**，该函数正确设置各个应用的状态后，直接通过调用应用所暴露出的生命周期钩子函数即可。当某个应用被推送到**appsToMount**后，它的**mount**函数会被调用，该应用就会被挂载；而推送到**appsToUnmount**中的应用则会调用其**unmount**钩子进行卸载。

### （3）应用入口

**single-spa**采用的是协议入口，即只要实现了**single-spa**的入口协议规范，它就是可加载的应用。**single-spa**的规范要求应用入口必须暴露出以下三个生命周期钩子函数，且必须返回Promise，以保证**single-spa**可以注册回调函数：

1. bootstrap
2. mount
3. unmount

![图片](https://uploader.shimo.im/f/qhvdFHRmJ47Ngyxj.png!thumbnail?fileGuid=cYYhDWjhK9qcKW6w)

**bootstrap**用于应用引导，基座应用会在子应用挂载前调用它。

**mount**用于应用挂载，就是一般应用中用于渲染的逻辑，即上述的new Vue语句。我们通常会把它封装到一个函数里，在mount钩子函数中调用。

**unmount**用于应用卸载，我们可以在这里调用实例的destroy方法手动卸载应用，或清除某些内存占用等。

除了以上三个必须实现的钩子外，**single-spa**还支持非必须的**load**、**unload**、**update**等，分别用于加载、卸载和更新应用。

### （4）应用加载

基于**system.js**如何启动**single-spa**：

```xml
<script type="systemjs-importmap">
  {
    "imports": {
      "app1": "http://localhost:8080/app1.js",
      "app2": "http://localhost:8081/app2.js",
      "single-spa": "https://cdnjs.cloudflare.com/ajax/libs/single-spa/4.3.7/system/single-spa.min.js"
    }
  }
</script>
... // system.js的相关依赖文件
<script>
(function(){
  // 加载single-spa
  System.import('single-spa').then((res)=>{
    var singleSpa = res;
    // 注册子应用
    singleSpa.registerApplication('app1',
      () => System.import('app1'),
      location => location.hash.startsWith(`#/app1`);
    );
    singleSpa.registerApplication('app2',
      () => System.import('app2'),
      location => location.hash.startsWith(`#/app2`);
    );
    // 启动single-spa
    singleSpa.start();
  })
})()
</script> 
```
在调用**singleSpa.registerApplication**注册应用时提供的第二个参数就是加载这个子应用的方法。
如果需要加载多个js，可以使用多个**System.import**连续导入。**single-spa**会调用这个函数，下载子应用代码并分别调用其**bootstrap**和**mount**方法进行引导和挂载。

从这里我们也可以看到**single-spa**的弊端。首先我们必须手动实现应用加载逻辑，挨个罗列子应用需要加载的资源，这在大型项目里是十分困难的（特别是使用了文件名hash时）；另外它只能以js文件为入口，无法直接以html为入口，这使得嵌入子应用变得很困难，也正因此，**single-spa**不能直接加载jQuery应用。

**single-spa**的**start**方法也很简单：

```javascript
export function start(opts) {
  started = true;
  if (opts && opts.urlRerouteOnly) {
    setUrlRerouteOnly(opts.urlRerouteOnly);
  }
  if (isInBrowser) {
    reroute();
  }
} 
```
先是设置**started**状态，然后设置我们上面说到的**urlRerouteOnly**属性，接着调用**reroute**，开始首次加载子应用。加载完第一个应用后，**single-spa**就时刻等待着**hashChange**或**popState**事件的触发，并执行应用的切换。
以上就是**single-spa**的核心原理，从上面的介绍中不难看出，single-spa只是负责把应用加载到一个页面中，至于应用能否协同工作，是很难保证的。而qiankun所要解决的，就是协同工作的问题。

### （5）应用示例

简单示例：

[https://zh-hans.single-spa.js.org/docs/getting-started-overview](https://zh-hans.single-spa.js.org/docs/getting-started-overview?fileGuid=cYYhDWjhK9qcKW6w)

vue示例：

[https://github.com/vue-microfrontends](https://github.com/vue-microfrontends?fileGuid=cYYhDWjhK9qcKW6w)

## 2、qiankun实现原理

### （1）应用加载

**single-spa**提供的应用加载方案是开放式的。针对上面的几个弊端，**qiankun**进行了一次封装，给出了一个更完整的应用加载方案，**qiankun**将其封装成了npm插件**import-html-entry**。

该方案的主要思路是允许以html文件为应用入口，然后通过一个html解析器从文件中提取js和css依赖，并通过fetch下载依赖，于是在**qiankun**中你可以这样配置入口：

```javascript
const MicroApps = [{
  name: 'app1',
  entry: 'http://localhost:8080',
  container: '#app',
  activeRule: '/app1'
}]
```
**qiankun**会通过**import-html-entry**请求**http://localhost:8080**，得到对应的html文件，解析内部的所有**script**和**style**标签，依次下载和执行它们，这使得应用加载变得更易用。
**import-html-entry**暴露出的核心接口是**importHTML**，用于加载html文件，它支持两个参数：

1. **url**，要加载的文件地址，一般是服务中html的地址
2. **opts**，配置参数

url不必多说。opts如果是一个函数，则会替换默认的fetch作为下载文件的方法，此时其返回值应当是Promise；如果是一个对象，那么它最多支持四个属性：**fetch**、**getPublicPath**、**getDomain**、**getTemplate**，用于替换默认的方法。

截取该函数的主要逻辑：

```javascript
export default function importHTML(url, opts = {}) {
  ...
  // 如果已经加载过，则从缓存返回，否则fetch回来并保存到缓存中
  return embedHTMLCache[url] || (embedHTMLCache[url] = fetch(url)
		.then(response => readResAsString(response, autoDecodeResponse))
		.then(html => {
		  // 对html字符串进行初步处理
		  const { template, scripts, entry, styles } = 
		    processTpl(getTemplate(html), assetPublicPath);
		  // 先将外部样式处理成内联样式
		  // 然后返回几个核心的脚本及样式处理方法
		  return getEmbedHTML(template, styles, { fetch }).then(embedHTML => ({
				template: embedHTML,
				assetPublicPath,
				getExternalScripts: () => getExternalScripts(scripts, fetch),
				getExternalStyleSheets: () => getExternalStyleSheets(styles, fetch),
				execScripts: (proxy, strictGlobal, execScriptsHooks = {}) => {
					if (!scripts.length) {
						return Promise.resolve();
					}
					return execScripts(entry, scripts, proxy, {
						fetch,
						strictGlobal,
						beforeExec: execScriptsHooks.beforeExec,
						afterExec: execScriptsHooks.afterExec,
					});
				},
			}));
		});
}
```
省略的部分主要是一些参数预处理，我们从return语句开始看，具体过程如下：
1. 检查是否有缓存，如果有，直接从缓存中返回
2. 如果没有，则通过fetch下载，并字符串化
3. 调用**processTpl**进行一次模板解析，主要任务是扫描出外联脚本和外联样式，保存在**scripts**和**styles**中
4. 调用**getEmbedHTML**，将外联样式下载下来，并替换到模板内，使其变成内部样式
5. 返回一个对象，该对象包含处理后的模板，以及**getExternalScripts**、**getExternalStyleSheets**、**execScripts**等几个核心方法。

![图片](https://uploader.shimo.im/f/JrPlRiJS31OrFCGq.png!thumbnail?fileGuid=cYYhDWjhK9qcKW6w)

**processTpl**主要基于正则表达式对模板字符串进行解析。我们来看**getExternalScripts**、**getExternalStyleSheets**、**execScripts**这三个方法：

**getExternalStyleSheets**

```javascript
export function getExternalStyleSheets(styles, fetch = defaultFetch) {
  return Promise.all(styles.map(styleLink => {
	if (isInlineCode(styleLink)) {
	  // if it is inline style
	  return getInlineCode(styleLink);
	} else {
	  // external styles
	  return styleCache[styleLink] ||
	  (styleCache[styleLink] = fetch(styleLink).then(response => response.text()));
	}
  ));
}
```
遍历styles数组，如果是内联样式，则直接返回；否则判断缓存中是否存在，如果没有，则通过fetch去下载，并进行缓存。
**getExternalScripts**与上述过程类似。

**execScripts**是实现js隔离的核心方法。

通过调用**importHTML**方法，**qiankun**可以直接加载html文件，同时将外联样式处理成内部样式表，并且解析出JavaScript依赖。更重要的是，它获得了一个可以在隔离环境下执行应用脚本的方法**execScripts**。

### （2）js隔离

**qiankun**通过**import-html-entry**，可以对html入口进行解析，并获得一个可以执行脚本的方法**execScripts**。**qiankun**引入该接口后，首先为该应用生成一个window的代理对象，然后将代理对象作为参数传入接口，以保证应用内的js不会对全局**window**造成影响。由于IE11不支持proxy，所以**qiankun**通过快照策略来隔离js，缺点是无法支持多实例场景。

我们先来看基于**proxy**的js隔离是如何实现的。首先看**import-html-entry**暴露出的接口，截取核心代码:

**execScripts**

```javascript
export function execScripts(entry, scripts, proxy = window, opts = {}) {
  ... // 初始化参数
  return getExternalScripts(scripts, fetch, error)
	.then(scriptsText => {
	  // 在proxy对象下执行脚本的方法
	  const geval = (scriptSrc, inlineScript) => {
	    const rawCode = beforeExec(inlineScript, scriptSrc) || inlineScript;
	    const code = getExecutableScript(scriptSrc, rawCode, proxy, strictGlobal);
        (0, eval)(code);
        afterExec(inlineScript, scriptSrc);
	  };
	  // 执行单个脚本的方法
      function exec (scriptSrc, inlineScript, resolve) { ... }
      // 排期函数，负责逐个执行脚本
      function schedule(i, resolvePromise) { ... }
      // 启动排期函数，执行脚本
      return new Promise(resolve => schedule(0, success || resolve));
    });
});
```
这个函数的关键是定义了三个函数：**geval**、**exec**、**schedule**，其中实现js隔离的是**geval**函数内调用的**getExecutableScript**函数。我们看到，在调这个函数时，我们把外部传入的**proxy**作为参数传入了进去，而它返回的是一串新的脚本字符串，这段新的字符串内的**window**已经被**proxy**替代，具体实现逻辑如下：
```javascript
function getExecutableScript(scriptSrc, scriptText, proxy, strictGlobal) {
	const sourceUrl = isInlineCode(scriptSrc) ? '' : `//# sourceURL=${scriptSrc}\n`;
	// 通过这种方式获取全局 window，因为 script 也是在全局作用域下运行的，所以我们通过 window.proxy 绑定时也必须确保绑定到全局 window 上
	// 否则在嵌套场景下， window.proxy 设置的是内层应用的 window，而代码其实是在全局作用域运行的，会导致闭包里的 window.proxy 取的是最外层的微应用的 proxy
	const globalWindow = (0, eval)('window');
	globalWindow.proxy = proxy;
	// TODO 通过 strictGlobal 方式切换切换 with 闭包，待 with 方式坑趟平后再合并
	return strictGlobal
		? `;(function(window, self, globalThis){with(window){;${scriptText}\n${sourceUrl}}}).bind(window.proxy)(window.proxy, window.proxy, window.proxy);`
		: `;(function(window, self, globalThis){;${scriptText}\n${sourceUrl}}).bind(window.proxy)(window.proxy, window.proxy, window.proxy);`;
}
```
### ![图片](https://uploader.shimo.im/f/gjqaIIvkkjaKXxqC.png!thumbnail?fileGuid=cYYhDWjhK9qcKW6w)

核心代码就是由两个矩形框起来的部分，它把解析出的**scriptText**（即脚本字符串）用**with(window){}**包裹起来，然后把**window.proxy**作为函数的第一个参数传进来，所以**with**语法内的**window**实际上是**window.proxy**。

这样，当在执行这段代码时，所有类似**var name = '张三'**这样的语句添加的全局变量**name**，实际上是被挂载到了**window.proxy**上，而不是真正的全局**window**上。当应用被卸载时，对应的**proxy**会被清除，因此不会导致js污染。而当你配置**webpack**的打包类型为**lib**时，你得到的接口大概如下：

```javascript
var jquery = (function(){})();
```
如果你的应用内使用了jquery，那么这个jquery对象就会被挂载到**window.proxy**上。不过如果你在代码内直接写**window.name = '张三'**来生成全局变量，那么**qiankun**就无法隔离js污染了。
**import-html-entry**实现了上述能力后，**qiankun**要做的就很简单了，只需要在加载一个应用时为其初始化一个**proxy**传递进来即可：

**proxySandbox.ts**

```javascript
export default class ProxySandbox implements SandBox {
  ...
  constructor(name: string) {
    ...
    const proxy = new Proxy(fakeWindow, {
      set () { ... },
      get () { ... }
    }
  }
}
```
每次加载一个应用，**qiankun**就初始化这样一个**proxySandbox**，传入上述**execScripts**函数中。
在IE下，由于**proxy**不被支持，并且没有可用的**polyfill**，所以**qiankun**退而求其次，采用快照策略实现js隔离。它的大致思路是，在加载应用前，将**window**上的所有属性保存起来（即拍摄快照）；等应用被卸载时，再恢复**window**上的所有属性，这样也可以防止全局污染。但是当页面同时存在多个应用实例时，**qiankun**无法将其隔离开，所以IE下的快照策略无法支持多实例模式。

### （3）css隔离

目前**qiankun**主要提供了两种样式隔离方案，一种是基于**shadowDom**的；另一种则是实验性的，思路类似于Vue中的**scoped**属性，给每个子应用的根节点添加一个特殊属性，用作对所有css选择器的约束。

开启样式隔离的语法如下：

```javascript
registerMicroApps({
  name: 'app1',
  ...
  sandbox: {
    strictStyleIsolation: true
    // 实验性方案，scoped方式
    // experimentalStyleIsolation: true
  },
})
```
当启用**strictStyleIsolation**时，**qiankun**将采用**shadowDom**的方式进行样式隔离，即为子应用的根节点创建一个**shadow root。**最终整个应用的所有DOM将形成一棵**shadow tree。**我们知道，**shadowDom**的特点是**，它**内部所有节点的样式对树外面的节点无效，因此自然就实现了样式隔离。
但是这种方案是存在缺陷的。因为某些UI框架可能会生成一些弹出框直接挂载到**document.body**下，此时由于脱离了**shadow tree**，所以它的样式仍然会对全局造成污染。

此外**qiankun**也在探索类似于**scoped**属性的样式隔离方案，可以通过**experimentalStyleIsolation**来开启。这种方案的策略是为子应用的根节点添加一个特定的随机属性，如：

```xml
<div
  data-qiankun-asiw732sde
  id="__qiankun_microapp_wrapper__"
  data-name="module-app1"
>
```
然后为所有样式前面都加上这样的约束：
```css
.app-main { 
  字体大小：14 px ; 
}
// ->
div[data-qiankun-asiw732sde] .app-main {  
  字体大小：14 px ; 
}
```
经过上述替换，这个样式就只能在当前子应用内生效了。虽然该方案已经提出很久了，但仍然是实验性的，因为它不支持@ keyframes，@ font-face，@ import，@ page（即不会被重写）。
### （4）应用通信

一般来说，微前端中各个应用之前的通信应该是尽量少的，而这依赖于应用的合理拆分。反过来说，如果你发现两个应用间存在极其频繁的通信，那么一般是拆分不合理造成的，这时往往需要将它们合并成一个应用。

当然了，应用间存在少量的通信是难免的。**qiankun**官方提供了一个简要的方案，思路是基于一个全局的**globalState**对象。这个对象由基座应用负责创建，内部包含一组用于通信的变量，以及两个分别用于修改变量值和监听变量变化的方法：**setGlobalState**和**onGlobalStateChange**。

以下代码用于在基座应用中初始化它：

```javascript
import { initGlobalState, MicroAppStateActions } from 'qiankun';
const initialState = {};
const actions: MicroAppStateActions = initGlobalState(initialState);
export default actions;
```
这里的**actions**对象就是我们说的**globalState**，即全局状态。基座应用可以在加载子应用时通过**props**将**actions**传递到子应用内，而子应用通过以下语句即可监听全局状态变化：
```javascript
actions.onGlobalStateChange (globalState, oldGlobalState) {
  ...
}
```
同样的，子应用也可以修改全局状态：
```javascript
actions.setGlobalState(...);
```
![图片](https://uploader.shimo.im/f/VaY36CMuSj29V037.png!thumbnail?fileGuid=cYYhDWjhK9qcKW6w)

此外，基座应用和其他子应用也可以进行这两个操作，从而实现对全局状态的共享，这样各个应用之间就可以通信了。这种方案与Redux和Vuex都有相似之处，只是由于微前端中的通信问题较为简单，所以官方只提供了这样一个精简方案。关于其实现原理这里不再赘述，感兴趣的可以去看一下源码。

# 二、qiankun应用

## 1、基础概念

## （1）基于路由配置

适用于 route-based 场景。

通过将微应用关联到一些 url 规则的方式，实现当浏览器 url 发生变化时，自动加载相应的微应用的功能。

### registerMicroApps(apps, lifeCycles?) 注册应用

* 参数
    * apps -******Array<RegistrableApp>**- 必选，微应用的一些注册信息
    * lifeCycles -**LifeCycles**- 可选，全局的微应用生命周期钩子
* 类型
    * **RegistrableApp**
        * name -**string**- 必选，微应用的名称，微应用之间必须确保唯一。
        * entry -**string | { scripts?: string[]; styles?: string[]; html?: string }**- 必选，微应用的入口。
            * 配置为字符串时，表示微应用的访问地址，例如**https://qiankun.umijs.org/guide/**。
            * 配置为对象时，**html**的值是微应用的 html 内容字符串，而不是微应用的访问地址。微应用的**publicPath**将会被设置为**/**。
        * container -**string | HTMLElement**- 必选，微应用的容器节点的选择器或者 Element 实例。如**container: '#root'**或**container: document.querySelector('#root')**。
        * activeRule -**string | (location: Location) => boolean | Array<string | (location: Location) => boolean>**- 必选，微应用的激活规则。
    * **LifeCycles 生命周期**
* 用法

注册微应用的基础配置信息。当浏览器 url 发生变化时，会自动检查每一个微应用注册的**activeRule**规则，符合规则的应用将会被自动激活。

* 示例
```javascript
import { registerMicroApps } from 'qiankun';
registerMicroApps(
  [
    {
      name: 'app1',
      entry: '//localhost:8080',
      container: '#container',
      activeRule: '/react',
      props: {
        name: 'kuitos',
      },
    },
  ],
  {
    beforeLoad: (app) => console.log('before load', app.name),
    beforeMount: [(app) => console.log('before mount', app.name)],
  },
);
```
### **start(opts?)**

* 用法

启动 qiankun。

* 示例
```javascript
import { start } from 'qiankun';
start();
```
## (2)手动加载微应用

适用于需要手动 加载/卸载 一个微应用的场景。

通常这种场景下微应用是一个不带路由的可独立运行的业务组件。 微应用不宜拆分过细，建议按照业务域来做拆分。业务关联紧密的功能单元应该做成一个微应用，反之关联不紧密的可以考虑拆分成多个微应用。 一个判断业务关联是否紧密的标准：**看这个微应用与其他微应用是否有频繁的通信需求**。如果有可能说明这两个微应用本身就是服务于同一个业务场景，合并成一个微应用可能会更合适。

### **loadMicroApp（app, configuration?）手动加载子应用**

* 参数
    * app 微应用基础信息
        * name
        * entry
        * container
        * props(可通过此参数传递主应用数据到子应用，包括vuex、router、全局方法，mixins等数据)
    * configuration  微应用配置信息
        * sandbox -**boolean | { strictStyleIsolation?: boolean, experimentalStyleIsolation?: boolean }**- 可选，是否开启沙箱，默认为**true**。
        * singular -**boolean | ((app: RegistrableApp<any>) => Promise<boolean>);**- 可选，是否为单实例场景，单实例指的是同一时间只会渲染一个微应用。默认为**false**。
        * fetch -**Function**- 可选，自定义的 fetch 方法。
        * getPublicPath -**(entry: Entry) => string**- 可选，参数是微应用的 entry 值。
        * getTemplate -**(tpl: string) => string**- 可选
        * excludeAssetFilter -**(assetUrl: string) => boolean**- 可选，指定部分特殊的动态加载的微应用资源（css/js) 不被 qiankun 劫持处理
* 返回值-**MicroApp**- 微应用实例
### **addGlobalUncaughtErrorHandler**

添加全局的未捕获异常处理器。

* 用法

手动加载一个微应用。

如果需要能支持主应用手动 update 微应用，需要微应用 entry 再多导出一个 update 钩子：

```javascript
export async function mount(props) {
  renderApp(props);
}
// 增加 update 钩子以便主应用手动更新微应用
export async function update(props) {
  renderPatch(props);
}
```
* 示例
```javascript
import { loadMicroApp } from 'qiankun';
import React from 'react';
class App extends React.Component {
  containerRef = React.createRef();
  microApp = null;
  componentDidMount() {
    this.microApp = loadMicroApp({
      name: 'app1',
      entry: '//localhost:1234',
      container: this.containerRef.current,
      props: { brand: 'qiankun' },
    });
  }
  componentWillUnmount() {
    this.microApp.unmount();
  }
  componentDidUpdate() {
    this.microApp.update({ name: 'kuitos' });
  }
  render() {
    return <div ref={this.containerRef}></div>;
  }
}
```
# 三、当前倍市得框架使用解析

当前倍市得采用**qiankun**框架，并不完全属于微前端概念，而是采用微组件，将主框架中组件拆分出来为全局组件，通过判断组件是否定制去选择加载主项目或者子项目中的组件，下面通过分析主项目，子项目去了解实现方式

## 1、主应用

### 技术解析：

*应用qiankun框架中的**loadMicroApp**即手动加载微应、**addGlobalUncaughtErrorHandler**

### 流程解析：

#### （1）组件拆分

将组件独立出来，将其中引用局部数据的地方修改为引用全局数据（以便拷贝代码到子项目时无需修改代码），将组件信息更新到全局组件维护文件micro.js中

![图片](https://uploader.shimo.im/f/8vka90IOSutmNiUH.png!thumbnail?fileGuid=cYYhDWjhK9qcKW6w)

#### （2）组件挂载

路由拦截中获取公司信息后，先对组件信息进行处理

![图片](https://uploader.shimo.im/f/8GrhGLoMrIvoXorj.png!thumbnail?fileGuid=cYYhDWjhK9qcKW6w)


## 2、子应用

### 技术解析：

子应用中主要根据qiankun中生命周期钩子函数去获取主应用数据，并进行组件注册等操作

```javascript
/**
 * bootstrap 只会在微应用初始化的时候调用一次，下次微应用重新进入时会直接调用 mount 钩子，不会再重复触发 bootstrap。
 * 通常我们可以在这里做一些全局变量的初始化，比如不会在 unmount 阶段被销毁的应用级别的缓存等。
 */
export async function bootstrap() {
  console.log('react app bootstraped');
}
/**
 * 应用每次进入都会调用 mount 方法，通常我们在这里触发应用的渲染方法
 */
export async function mount(props) {}
/**
 * 应用每次 切出/卸载 会调用的方法，通常在这里我们会卸载微应用的应用实例
 */
export async function unmount(props) {}
/**
 * 可选生命周期钩子，仅使用 loadMicroApp 方式加载微应用时生效
 */
export async function update(props) {
  console.log('update props', props);
}
```
### 流程解析：

#### （1）组件注册

![图片](https://uploader.shimo.im/f/WlayGatXnMjUgPRp.png!thumbnail?fileGuid=cYYhDWjhK9qcKW6w)

#### （2）数据处理

* 继承主应用传递的vuex数据，与子应用vuex数据进行合并，同名数据覆盖
## 3、主子项目通信

* 子应用mount生命周期内对拿到的vuex数据与子应用合并
* 主应用将Vue实例、utils通用项目工具函数、api通用项目api、global全局枚举数据放置在mainAppData中传递给子组件，子组件在mount中接收到后与本地数据合并
## 4、脚手架


