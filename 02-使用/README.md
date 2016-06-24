# webpack 初探

## webpack 机制
  
webpack 的机制很简单，通过指定一个入口文件，从入口文件进行静态依赖分析，将所有依赖打包到一起，产生一个最终的输出文件。

真正让 webpack 变得无比强大的原因，是它的 [Loaders](https://github.com/liximomo/webpack-guide/blob/master/Loaders/README.md) 和 [Code Splitting](https://github.com/liximomo/webpack-guide/blob/master/Code%Splitting/README.md)机制。

webpack 默认使用 [CommonJS](https://github.com/liximomo/webpack-guide/blob/master/CommonJS/README.md) 模块语法，这使得 webpack 可以直接使用 npm 上数十万的库。

## 示例

首先创建一个静态页面 index.html 和一个 JS 入口文件 entry.js：

```html
<!-- index.html -->
<html>
<head>
  <meta charset="utf-8">
</head>
<body>
  <div id="foo"></div>
  <script src="output.bundle.js"></script>
</body>
</html>
```

```js
// entry.js
document.getElementById('foo').innerHTML = 'Hello webpack!';
```

然后编译 entry.js 并打包到 bundle.js，我们需要告诉 webpack 一个入口文件，并制定输出文件名：

```bash
$ webpack entry.js output.bundle.js
```


打包过程会显示日志：

```
Hash: fa58c03092b2cde2a45e
Version: webpack 1.13.1
Time: 58ms
           Asset     Size  Chunks             Chunk Names
output.bundle.js  1.45 kB       0  [emitted]  main
   [0] ./entry.js 61 bytes {0} [built]
```

用浏览器打开 `index.html` 将会看到 `Hello webpack!` 。

接下来添加一个模块 `a.js` 和 `a-1.js` 并修改入口 `entry.js`：

```js
// a-1.js
module.exports = '来自 a-1 的消息';
```

```js
// a.js
module.exports = require('./a-1.js') + '<br />' + '来自 a 的消息.';
```

```js
// entry.js
document.getElementById('foo').innerHTML = 'Hello webpack!' + '<br />' + require('./a.js');
```

重新打包 `webpack entry.js output.bundle.js` 后刷新页面看到来自模块 a 和 a-1 的信息。

```
Hash: f04ab7771dc4b9a9607f
Version: webpack 1.13.1
Time: 62ms
           Asset     Size  Chunks             Chunk Names
output.bundle.js  1.73 kB       0  [emitted]  main
   [0] ./entry.js 91 bytes {0} [built]
   [1] ./a.js 63 bytes {0} [built]
   [2] ./a-1.js 31 bytes {0} [built]
```

Webpack 会分析入口文件，解析包含依赖关系的各个文件。这些文件（模块）都打包到 bundle.js 。Webpack 会给每个模块分配一个唯一的 id 并通过这个 id 索引和访问模块。在页面启动时，会先执行 entry.js 中的代码，其它模块会在运行 `require` 的时候再执行。

## 总结

webpack 使用 CommonJS 简洁易懂的语法，借助于 npm 庞大的开源库数量，以及非常简单的使用方法，使得前端模块从未如此简单，接下来会介绍 webpack 的一些强大功能。

## 示例运行

### 环境请求

nodejs v0.6以上，建议安装最新版本。

### 运行

  1. 克隆本仓库到本地，进入此目录
  2. `npm install` 

     这条命令会为当前目录安装 webpack 依赖

  3. `npm start`

  
这条命令会运行预先定义好的脚本。等价于 webpack entry.js output.bundle.js 
(由于 npm 安装 webpack 时使用的本地安装，所以 webpack 命令并不在 PATH 环境变量中，bash 中无法识别 webpack 命令，所以要借助于 npm 运行 webpack 命令)

关于 node 的包管理系统 npm 不必深究，只要知道它是一个包管理器即可。

