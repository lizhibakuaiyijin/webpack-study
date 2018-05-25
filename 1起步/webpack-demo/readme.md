#Webpack4
官网链接：https://www.webpackjs.com/guides/getting-started/
## 安装
前提 安装`node.js`

###本地安装
* 安装最新版本	
 
```
npm install --save-dev webpack
```
* 安装特定版本

```
npm install --save-dev webpack@<version>
```
* 使用 `webpack 4+` 版本 还需安装CLI

```
npm install --save-dev webpack-cli
```

对于大多数项目，我们建议本地安装。这可以使我们在引入破坏式变更(breaking change)的依赖时，更容易分别升级项目。通常，`webpack` 通过运行一个或多个 `npm scripts`，会在本地 `node_modules` 目录中查找安装的`webpack`：
```
"scripts": {
    "start": "webpack --config webpack.config.js"
}
```

###全局安装
```
npm install --global webpack
```

##起步
###基本安装
首先我们创建一个目录，初始化 `npm`，然后 在本地安装 `webpack`，接着安装 `webpack-cli`（此工具用于在命令行中运行 `webpack`）：
```
mkdir webpack-demo && cd webpack-demo
sudo cnpm init -y
sudo cnpm install webpack webpack-cli --save-dev
```
创建以下目录结构、文件和内容
project
```
  webpack-demo
  |- package.json
+ |- index.html
+ |- /src
+   |- index.js
```
index.html
```
<!doctype html>
<html>
  <head>
    <title>起步</title>
    <script src="https://unpkg.com/lodash@4.16.6"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```
src/index.js
```
function component() {
  var element = document.createElement('div');

  // Lodash（目前通过一个 script 脚本引入）对于执行这一行是必需的
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

调整 `package.json` 文件，以便确保我们安装包是私有的(private)，并且移除 `main` 入口。这可以防止意外发布你的代码。
package.json
```
 {
    "name": "webpack-demo",
    "version": "1.0.0",
    "description": "",
+   "private": true,
-   "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "webpack": "^4.0.1",
      "webpack-cli": "^2.0.9"
    },
    "dependencies": {}
  }

```

在此示例中，`<script>` 标签之间存在隐式依赖关系。`index.js` 文件执行之前，还依赖于页面中引入的 `lodash`。之所以说是隐式的是因为 `index.js` 并未显式声明需要引入 `lodash`，只是假定推测已经存在一个全局变量 `_`。


使用这种方式去管理 `JavaScript` 项目会有一些问题：

* 无法立即体现，脚本的执行依赖于外部扩展库(external library)。
* 如果依赖不存在，或者引入顺序错误，应用程序将无法正常运行。
* 如果依赖被引入但是并没有使用，浏览器将被迫下载无用代码。
让我们使用 `webpack` 来管理这些脚本。


### 创建一个bundle文件
首先，我们稍微调整下目录结构，将“源”代码(`/src`)从我们的“分发”代码(`/dist`)中分离出来。“源”代码是用于书写和编辑的代码。“分发”代码是构建过程产生的代码最小化和优化后的“输出”目录，最终将在浏览器中加载：
project
```
  webpack-demo
  |- package.json
+ |- /dist
+   |- index.html
- |- index.html
  |- /src
    |- index.js
```
要在 `index.js` 中打包 `lodash` 依赖，我们需要在本地安装 `library`：
```
sudo cnpm install --save lodash
```

==在安装一个要打包到生产环境的安装包时，你应该使用 `npm install --save`，如果你在安装一个用于开发环境的安装包（例如，linter, 测试库等），你应该使用 `npm install --save-dev`。==（？）

src/index.js
```
+ import _ from 'lodash';
+ 
  function component() {
    var element = document.createElement('div');

-   // Lodash, currently included via a script, is required for this line to work
+   // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');

    return element;
  }

  document.body.appendChild(component());
```

现在，由于通过打包来合成脚本，我们必须更新 `index.html` 文件。因为现在是通过 `import` 引入 `lodash`，所以将 `lodash <script>` 删除，然后修改另一个 `<script>` 标签来加载 `bundle`，而不是原始的 `/src` 文件：

dist/index.html
```
  <!doctype html>
  <html>
   <head>
     <title>起步</title>
-    <script src="https://unpkg.com/lodash@4.16.6"></script>
   </head>
   <body>
-    <script src="./src/index.js"></script>
+    <script src="main.js"></script>
   </body>
  </html>
```
在这个设置中，`index.js` 显式要求引入的 `lodash` 必须存在，然后将它绑定为 `_`（没有全局作用域污染）。通过声明模块所需的依赖，`webpack` 能够利用这些信息去构建依赖图，然后使用图生成一个优化过的，会以正确顺序执行的 `bundle`。
可以这样说，执行 `npx webpack`，会将我们的脚本作为入口起点，然后 输出 为 `main.js`。`Node 8.2+` 版本提供的 `npx` 命令，可以运行在初始安装的 `webpack` 包(`package`)的 `webpack` 二进制文件（`./node_modules/.bin/webpack`）：
```
npx webpack
Hash: b1f3d498c93956cdeec9
Version: webpack 4.8.3
Time: 3492ms
Built at: 2018-05-23 14:45:14
  Asset    Size  Chunks             Chunk Names
main.js  70 KiB       0  [emitted]  main
Entrypoint main = main.js
[1] (webpack)/buildin/module.js 519 bytes {0} [built]
[2] (webpack)/buildin/global.js 509 bytes {0} [built]
[3] ./src/index.js 251 bytes {0} [built]
    + 1 hidden module

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
###模块
`ES2015` 中的 `import` 和 `export` 语句已经被标准化。虽然大多数浏览器还无法支持它们，但是 `webpack` 却能够提供开箱即用般的支持。
事实上，`webpack` 在幕后会将代码“转译”，以便旧版本浏览器可以执行。如果你检查 `dist/bundle.js`，你可以看到 `webpack` 具体如何实现，这是独创精巧的设计！除了 `import` 和 `export`，`webpack` 还能够很好地支持多种其他模块语法，更多信息请查看模块 API。
注意，`webpack` 不会更改代码中除 `import` 和 `export` 语句以外的部分。如果你在使用其它 `ES2015` 特性，请确保你在 `webpack` 的 `loader` 系统中使用了一个像是 `Babel` 或 `Bublé` 的转译器。
###使用一个配置文件
在 `webpack 4` 中，可以无须任何配置使用，然而大多数项目会需要很复杂的设置，这就是为什么 `webpack` 仍然要支持 配置文件。这比在终端(terminal)中手动输入大量命令要高效的多，所以让我们创建一个取代以上使用 `CLI` 选项方式的配置文件：
project
```
 webpack-demo
  |- package.json
+ |- webpack.config.js
  |- /dist
    |- index.html
  |- /src
    |- index.js

```
webpack.config.js
```
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

通过新配置再次执行构建：
```
npx webpack --config webpack.config.js
```
```
Hash: 1e97362c455a4dc97d84
Version: webpack 4.8.3
Time: 3778ms
Built at: 2018-05-23 14:55:17
    Asset    Size  Chunks             Chunk Names
bundle.js  70 KiB       0  [emitted]  main
Entrypoint main = bundle.js
[1] (webpack)/buildin/module.js 519 bytes {0} [built]
[2] (webpack)/buildin/global.js 509 bytes {0} [built]
[3] ./src/index.js 251 bytes {0} [built]
    + 1 hidden module

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
如果 `webpack.config.js` 存在，则 `webpack` 命令将默认选择使用它。我们在这里使用 `--config` 选项只是向你表明，可以传递任何名称的配置文件。这对于需要拆分成多个文件的复杂配置是非常有用。

比起 `CLI` 这种简单直接的使用方式，配置文件具有更多的灵活性。我们可以通过配置方式指定 `loader` 规则(`loader rules`)、插件(`plugins`)、解析选项(`resolve options`)，以及许多其他增强功能。


###NPM 脚本(NPM Scripts)
考虑到用 `CLI` 这种方式来运行本地的 `webpack` 不是特别方便，我们可以设置一个快捷方式。在 `package.json` 添加一个 `npm` 脚本(`npm script`)：

package.json
```
  {
    "name": "webpack-demo",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
+     "build": "webpack"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "webpack": "^4.0.1",
      "webpack-cli": "^2.0.9",
      "lodash": "^4.17.5"
    }
  }
```
现在，可以使用 `npm run build` 命令，来替代我们之前使用的 `npx` 命令。注意，使用 `npm` 的 `scripts`，我们可以像使用 `npx` 那样通过模块名引用本地安装的 npm 包。这是大多数基于 `npm` 的项目遵循的标准，因为它允许所有贡献者使用同一组通用脚本（如果必要，每个 `flag` 都带有 `--config` 标志）。
现在运行以下命令，然后看看你的脚本别名是否正常运行：
```
npm run build

> webpack-demo@1.0.0 build /Applications/XAMPP/xamppfiles/htdocs/FrontEnd/2018/May/webpack-demo
> webpack

Hash: 1e97362c455a4dc97d84
Version: webpack 4.8.3
Time: 3704ms
Built at: 2018-05-23 15:04:26
    Asset    Size  Chunks             Chunk Names
bundle.js  70 KiB       0  [emitted]  main
Entrypoint main = bundle.js
[1] (webpack)/buildin/module.js 519 bytes {0} [built]
[2] (webpack)/buildin/global.js 509 bytes {0} [built]
[3] ./src/index.js 251 bytes {0} [built]
    + 1 hidden module

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
通过向 `npm run build` 命令和你的参数之间添加两个中横线，可以将自定义参数传递给 `webpack`，例如：`npm run build -- --colors`。

###结论
本章实现了一个基本的构建过程，接下来了解如何通过 webpack 来管理资源，例如图片、字体。

