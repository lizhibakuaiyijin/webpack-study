#Webpack4
官网链接：https://www.webpackjs.com/guides/getting-started/
## 管理输出
### 上一节代码回退
删除图片、字体、`.css`、`xml`文件及相关代码和配置

到目前为止，我们在 `index.html` 文件中手动引入所有资源，然而随着应用程序增长，并且一旦开始对文件名使用哈希(hash)]并输出多个 bundle，手动地对 `index.html` 文件进行管理，一切就会变得困难起来。然而，可以通过一些插件，会使这个过程更容易操控。

###预先准备
project
```
  webpack-demo
  |- package.json
  |- webpack.config.js
  |- /dist
  |- /src
    |- index.js
+   |- print.js
  |- /node_modules
```
src/print.js
```
export default function printMe() {
  console.log('I get called from print.js!');
}
```

更新 `dist/index.html` 文件，来为 webpack 分离入口做好准备：


现在调整配置。我们将在 entry 添加 `src/print.js` 作为新的入口起点（`print`），然后修改 output，以便根据入口起点名称动态生成 bundle 名称：
webpack.config.js
```
  const path = require('path');

  module.exports = {
-   entry: './src/index.js',
+   entry: {
+     app: './src/index.js',
+     print: './src/print.js'
+   },
    output: {
-     filename: 'bundle.js',
+     filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```
构建
```
npm run build

> webpack-demo@1.0.0 build /Applications/XAMPP/xamppfiles/htdocs/webpack-study/AssetOutput/webpack-demo
> webpack

Hash: 64b657fe15e9e74976ec
Version: webpack 4.8.3
Time: 3681ms
Built at: 2018-05-24 09:25:49
          Asset       Size  Chunks             Chunk Names
print.bundle.js  660 bytes       0  [emitted]  print
  app.bundle.js   70.2 KiB    1, 0  [emitted]  app
Entrypoint app = app.bundle.js
Entrypoint print = print.bundle.js
[0] ./src/print.js 83 bytes {0} {1} [built]
[2] (webpack)/buildin/module.js 519 bytes {1} [built]
[3] (webpack)/buildin/global.js 509 bytes {1} [built]
[4] ./src/index.js 435 bytes {1} [built]
    + 1 hidden module

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```

我们可以看到，webpack 生成 `print.bundle.js` 和 `app.bundle.js` 文件，这也和我们在 `index.html` 文件中指定的文件名称相对应。如果你在浏览器中打开 `index.html`，就可以看到在点击按钮时会发生什么。
但是，如果我们更改了我们的一个入口起点的名称，甚至添加了一个新的名称，会发生什么？生成的包将被重命名在一个构建中，但是我们的`index.html`文件仍然会引用旧的名字。我们用 ==HtmlWebpackPlugin== 来解决这个问题。


###设定 HtmlWebpackPlugin
首先安装插件，并且调整 `webpack.config.js` 文件：
```
sudo cnpm install --save-dev html-webpack-plugin
```
webpack.config.js
```
 const path = require('path');
+ const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
+   plugins: [
+     new HtmlWebpackPlugin({
+       title: 'Output Management'
+     })
+   ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```
在我们构建之前，你应该了解，虽然在 `dist/` 文件夹我们已经有 `index.html` 这个文件，然而 ==HtmlWebpackPlugin== 还是会默认生成 `index.html` 文件。这就是说，它会用新生成的 `index.html` 文件，把我们的原来的替换。让我们看下在执行 `npm run build` 后会发生什么：

在这里发生了一个问题,执行完上述操作之后，我并没有成功的构建，报错了。查看`package.json`发现 没有安装`webpack`和`webpack-cli`
```
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "lodash": "^4.17.10"
  },
  "devDependencies": {
    "css-loader": "^0.28.11",
    "csv-loader": "^2.1.1",
    "file-loader": "^1.1.11",
    "style-loader": "^0.21.0",
    "xml-loader": "^1.2.1"
  }
}
```
终端也有提示，只是我忽略了....（小疑问：`file-loader`也需要安装webpack，为什么上一节构建的时候没有报错？）
```
npm install
npm WARN file-loader@1.1.11 requires a peer of webpack@^2.0.0 || ^3.0.0 || ^4.0.0 but none is installed. You must install peer dependencies yourself.
npm WARN html-webpack-plugin@3.2.0 requires a peer of webpack@^1.0.0 || ^2.0.0 || ^3.0.0 || ^4.0.0 but none is installed. You must install peer dependencies yourself.
npm WARN schema-utils@0.4.5 requires a peer of webpack@^2.0.0 || ^3.0.0 || ^4.0.0 but none is installed. You must install peer dependencies yourself.

up to date in 1.205s
```
成功构建~
```
npm run build

> webpack-demo@1.0.0 build /Applications/XAMPP/xamppfiles/htdocs/webpack-study/AssetOutput/webpack-demo
> webpack

Hash: 29c063a4e944e8d1f1c5
Version: webpack 4.8.3
Time: 545ms
Built at: 2018-05-25 10:10:20
          Asset       Size  Chunks             Chunk Names
print.bundle.js  660 bytes       0  [emitted]  print
  app.bundle.js   70.2 KiB    1, 0  [emitted]  app
     index.html  254 bytes          [emitted]  
Entrypoint app = app.bundle.js
Entrypoint print = print.bundle.js
[0] ./src/print.js 83 bytes {0} {1} [built]
[2] (webpack)/buildin/module.js 519 bytes {1} [built]
[3] (webpack)/buildin/global.js 509 bytes {1} [built]
[4] ./src/index.js 435 bytes {1} [built]
    + 1 hidden module
```
如果你在代码编辑器中将 `index.html` 打开，你就会看到 `HtmlWebpackPlugin` 创建了一个全新的文件，所有的 bundle 会自动添加到 html 中。


###清理 /dist 文件夹
你可能已经注意到，由于过去的指南和代码示例遗留下来，导致我们的 `/dist`文件夹相当杂乱。webpack 会生成文件，然后将这些文件放置在 /dist 文件夹中，但是 webpack 无法追踪到哪些文件是实际在项目中用到的。
通常，在每次构建前清理 `/dist` 文件夹，是比较推荐的做法，因此只会生成用到的文件。让我们完成这个需求。
`clean-webpack-plugin` 是一个比较普及的管理插件，让我们安装和配置下。
```
npm install clean-webpack-plugin --save-dev
```
webpack.config.js
```
 const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
+ const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
    plugins: [
+     new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Output Management'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```
现在执行 `npm run build`，再检查 `/dist` 文件夹。如果一切顺利，你现在应该不会再看到旧的文件，只有构建后生成的文件！
###结论

已经了解如何向 HTML 动态添加 bundle 和清理dist文件夹~

