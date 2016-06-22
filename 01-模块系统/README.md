# 模块系统

现今网站已经进化成为了 web app:

* 越来越多的 JavaScript 被使用。
* 现代浏览器提供了广泛的接口。
* 越来越少的整页刷新

结果就是客户端的代码**越来越多** 。

大量的代码基要被管理。模块系统使我们可以将代码基分割为模块。

# 模块系统样式

目前存在多种定义依赖和到处值得标准。

* `<script>`-标签样式 (不适用模块系统)
* CommonJS
* AMD and some dialects of it
* ES6 modules
* 更多...

## `<script>`-标签样式

这是不使用模块系统时你会使用的方式。

``` html
<script src="module1.js"></script>
<script src="module2.js"></script>
<script src="libraryA.js"></script>
<script src="module3.js"></script>
```

这种模块导出一个接口到全局变量，也就是 `window` 对象。模块可以访问全局对象上依赖的接口。

#### 问题

* 全局对象上存在冲突。
* 需要精确控制加载顺序。
* 开发者自己解决依赖。
* 大项目中加载列表会很长很难管理。

## CommonJs: 同步 `require`

这种形式使用同步的 `require` 方法去加载依赖并返回导出的借口。一个模块可以通过为 `exports` 添加属性或为 `module.exports` 赋值来到指定导出。

``` javascript
require("module");
require("../file.js");
exports.doStuff = function() {};
module.exports = someValue;
```

这种模块系统被 [node.js](http://nodejs.org) 使用在服务器端.

#### 优点

* 服务器端模块可以被重用
* [npm](https://www.npmjs.com/) 上存在大量可用的模块 
* 简单易用

#### 缺点

* 在网络请求上使用阻塞调用效果并不好，因为网络请求是异步的。
* 不能并行请求多个模块

#### 实现

* [node.js](http://nodejs.org/) - 服务端
* [browserify](https://github.com/substack/node-browserify)
* [modules-webmake](https://github.com/medikoo/modules-webmake) - 静态编译打包
* [wreq](https://github.com/substack/wreq) - 客户端

## AMD: 异步模块请求

[`Asynchronous Module Definition`](https://github.com/amdjs/amdjs-api/wiki/AMD)

其他模块系统(浏览器端)在处理同步的 `require` (CommonJS) 会有问题，所有引入了一个异步版本(一种新的方式定义依赖和导出):

``` javascript
require(["module", "../file"], function(module, file) { /* ... */ });
define("mymodule", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
```

#### 优点

* 与网络的异步性相符合。
* 并行请求多个模块。

#### 缺点

* 更多额外的代码.更难越多和编写。
* 看起来是某种变通方案。

#### 实现

* [require.js](http://requirejs.org/) - 客户端
* [curl](https://github.com/cujojs/curl) - 客户端

了解更多 [CommonJS]() and [AMD]().

## ES6 模块

EcmaScript6 为 JavaScript 添加了一些语言构建, 引入了一种新的模块系统.

``` javascript
import "jquery";
export function doStuff() {}
module "localModule" {}
```

#### 优点

* 静态分析，简单。
* ES 的标准.

#### 缺点

* 本地浏览器支持还需要写时间。
* 使用这种语法的模块还不太多。

## 期望的模块系统

可以兼容多种模块风格，可以利用已有的代码，可以方便的定制模块系统。

---

# 传输

模块应该并运行在客户端，所以必须可以从服务端传输到浏览器。

有两种极端的传输模块的方式。

* 每个模块一个请求
* 所有模块一个请求

这两种方式都是次优的:

* 每个模块一个请求
    * 优点: 只有请求的模块会被传输
    * 缺点: 过多的请求意味着过多不必要的网络负载。
    * 缺点: 由于请求的延迟，所以应用启动速度慢
* 所有模块一个请求
    * 优点: 更少的不必要的网络复杂，更少的延迟
    * 缺点: 当前未请求的模块也会被传输。

## Chunked 传输

一个更弹性的传输方式会更好。大多数情况下我们都需要一个更中和的方式。

→ 当编译所有模块时: 将模块的集合分割为多个小的chunks(没有好的中文译名，可以理解为一堆东西捆绑在一起形成的一个快).

这种方式允许多个较小的，较快的请求。初始没有被请求的模块 chunks 可以在后续按需请求。这可以加速网页的初始加载时间并且后续仍然可以按需加载额外的 chunk。 

这种 "split points(代码分割点)" 完全有开发者来决定。

→ 现在我们可以处理更复杂的代码基了。

注意: 这个方式是来自 [Google's GWT](https://developers.google.com/web-toolkit/doc/latest/DevGuideCodeSplitting). 

了解 [Code Splitting]().

---

# 为什么只是JavaScript?

为什么模块系统只是帮助 javascript 开发者? 还有许多其他的资源需要被处理：

* stylesheets
* images
* webfonts
* html for templating
* etc.

Or 转译/处理:

* coffeescript → javascript
* elm → javascript
* less/sass stylesheets → css stylesheets
* jade templates → javascript which generates html
* i18n files → something
* etc.

这些应该下 javascript 一样简单：

``` javascript
require("./style.css");
```

``` javascript
require("./style.less");
require("./template.jade");
require("./image.png");
```

了解 [Using loaders]() and [Loaders]() .

---

# 静态分析

当编译所有模块时，静态分析试图查找所有依赖。

传统的模块系统不能解析使用了表达式的依赖资源，但是 `require("./template/" + templateName + ".jade")` 形式的请求是种很常见的需求，比如 app 路由。  

## 策略

一个聪明的解析器允许存在的大部分代码运行。如果开发者做了怪异的事情，它会试图找出最协调的解决方案。

# webpack

webpack 作为静态资源依赖管理的解决方案，使用 chunked 传输，表达式依赖解析，兼容所有模块系统，管理任意静态资源，支持 transpiler，使我们可以管理非常复杂的代码基。
