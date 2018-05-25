#Webpack4
官网链接：https://www.webpackjs.com/guides/getting-started/
## 开发
沿用上一节代码~
###使用 source map
当 webpack 打包源代码时，可能会很难追踪到错误和警告在源代码中的原始位置。例如，如果将三个源文件（`a.js`, `b.js` 和 `c.js`）打包到一个 bundle（`bundle.js`）中，而其中一个源文件包含一个错误，那么堆栈跟踪就会简单地指向到 `bundle.js`。这并通常没有太多帮助，因为你可能需要准确地知道错误来自于哪个源文件。
为了更容易地追踪错误和警告，JavaScript 提供了 source map 功能，将编译后的代码映射回原始源代码。如果一个错误来自于 `b.js`，==source map== 就会明确的告诉你。
==source map== 有很多不同的选项可用，请务必仔细阅读它们，以便可以根据需要进行配置。
对于本指南，我们使用 `inline-source-map` 选项，这有助于解释说明我们的目的（仅解释说明，不要用于生产环境）：
webpack.config.js
```
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
+   devtool: 'inline-source-map',
    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Development'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```
现在，让我们来做一些调试，在 `print.js` 文件中生成一个错误：
print.js
```
  export default function printMe() {
-   console.log('I get called from print.js!');
+   cosnole.error('I get called from print.js!');
  }
```
构建~
点击按钮，控制台报错：
```
print.js:3 Uncaught ReferenceError: cosnole is not defined
    at HTMLButtonElement.e (print.js:3)
```
此错误包含有发生错误的文件（`print.js`）和行号（3）的引用。这是非常有帮助的，因为现在我们知道了，所要解决的问题的确切位置。
###使用观察模式
你可以指示 webpack "watch" 依赖图中的所有文件以进行更改。如果其中一个文件被更新，代码将被重新编译，所以你不必手动运行整个构建。
我们添加一个用于启动 webpack 的观察模式的 npm script 脚本：
package.json
```
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
+   "watch": "webpack --watch",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "lodash": "^4.17.10"
  },
  "devDependencies": {
    "clean-webpack-plugin": "^0.1.19",
    "css-loader": "^0.28.11",
    "csv-loader": "^2.1.1",
    "file-loader": "^1.1.11",
    "html-webpack-plugin": "^3.2.0",
    "style-loader": "^0.21.0",
    "webpack": "^4.8.3",
    "webpack-cli": "^2.1.4",
    "xml-loader": "^1.2.1"
  }
}
```
现在，你可以在命令行中运行 `npm run watch`，就会看到 webpack 编译代码，然而却不会退出命令行。这是因为 script 脚本还在观察文件。
现在，webpack 观察文件的同时，我们先移除我们之前引入的错误：
src/print.js
```
export default function printMe() {
  console.log('I get called from print.js!');
  // cosnole.error('I get called from print.js!');
}
```
现在,保存文件并检查终端窗口。应该可以看到 webpack 自动重新编译修改后的模块！
唯一的缺点是，为了看到修改后的实际效果，你需要刷新浏览器。如果能够自动刷新浏览器就更好了，可以尝试使用 `webpack-dev-server`，恰好可以实现我们想要的功能。

###使用 webpack-dev-server
`webpack-dev-server` 为你提供了一个简单的 web 服务器，并且能够实时重新加载(live reloading)。让我们设置以下：
```
npm install --save-dev webpack-dev-server
```
修改配置文件，告诉开发服务器(dev server)，在哪里查找文件：
webpack.config.js
```
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
    devtool: 'inline-source-map',
+   devServer: {
+     contentBase: './dist'
+   },
    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Development'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```
以上配置告知 `webpack-dev-server`，在 localhost:8080 下建立服务，将 dist 目录下的文件，作为可访问文件。
让我们添加一个 script 脚本，可以直接运行开发服务器(dev server)：
package.json
```
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "watch": "webpack --watch",
+   "start": "webpack-dev-server --open",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "lodash": "^4.17.10"
  },
  "devDependencies": {
    "clean-webpack-plugin": "^0.1.19",
    "css-loader": "^0.28.11",
    "csv-loader": "^2.1.1",
    "file-loader": "^1.1.11",
    "html-webpack-plugin": "^3.2.0",
    "style-loader": "^0.21.0",
    "webpack": "^4.8.3",
    "webpack-cli": "^2.1.4",
    "webpack-dev-server": "^3.1.4",
    "xml-loader": "^1.2.1"
  }
}
```
现在，我们可以在命令行中运行 `npm start`，就会看到浏览器自动加载页面。如果现在修改和保存任意源文件，web 服务器就会自动重新加载编译后的代码。试一下！

###使用 webpack-dev-middleware
`webpack-dev-middleware` 是一个容器(wrapper)，它可以把 webpack 处理后的文件传递给一个服务器(server)。 `webpack-dev-server` 在内部使用了它，同时，它也可以作为一个单独的包来使用，以便进行更多自定义设置来实现更多的需求。接下来是一个 webpack-dev-middleware 配合 express server 的示例。
首先，安装 `express` 和 `webpack-dev-middleware`：
```
npm install --save-dev express webpack-dev-middleware
```
接下来我们需要对 webpack 的配置文件做一些调整，以确保中间件(middleware)功能能够正确启用：
webpack.config.js
```
const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
    devtool: 'inline-source-map',
    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Output Management'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist'),
+     publicPath: '/'
    }
  };
```
publicPath 也会在服务器脚本用到，以确保文件资源能够在 http://localhost:3000 下正确访问，我们稍后再设置端口号。下一步就是设置我们自定义的 express 服务：
project
```
  webpack-demo
  |- package.json
  |- webpack.config.js
+ |- server.js
  |- /dist
  |- /src
    |- index.js
    |- print.js
  |- /node_modules
```
server.js
```
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');

const app = express();
const config = require('./webpack.config.js');
const compiler = webpack(config);

// Tell express to use the webpack-dev-middleware and use the webpack.config.js
// configuration file as a base.
app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath
}));

// Serve the files on port 3000.
app.listen(3000, function () {
  console.log('Example app listening on port 3000!\n');
});
```
现在，添加一个 npm script，以使我们更方便地运行服务：
```
"server": "node server.js",
```
构建

打开浏览器，跳转到 http://localhost:3000，看到你的webpack 应用程序已经运行！
###结论
学会了如何自动编译代码，并运行一个简单的开发服务器(development server)。


