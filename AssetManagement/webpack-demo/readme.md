#Webpack4
官网链接：https://www.webpackjs.com/guides/getting-started/
## 管理资源
`webpack` 最出色的功能之一就是，除了 `JavaScript`，还可以通过 `loader` 引入任何其他类型的文件。也就是说，以上列出的那些 JavaScript 的优点（例如显式依赖），同样可以用来构建网站或 web 应用程序中的所有非 JavaScript 内容。
###安装
###加载css
为了从 `JavaScript` 模块中 `import` 一个 `CSS` 文件，你需要在 `module` 配置中 安装并添加 `style-loader` 和 `css-loader`：
```
sudo cnpm install --save-dev style-loader css-loader
```
webpack.config.js
```
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
+   module: {
+     rules: [
+       {
+         test: /\.css$/,
+         use: [
+           'style-loader',
+           'css-loader'
+         ]
+       }
+     ]
+   }
  };

```
`webpack` 根据正则表达式，来确定应该查找哪些文件，并将其提供给指定的 `loader`。在这种情况下，以 `.css` 结尾的全部文件，都将被提供给 `style-loader` 和 `css-loader`。

这使你可以在依赖于此样式的文件中` import './style.css'`。现在，当该模块运行时，含有 `CSS` 字符串的 `<style>` 标签，将被插入到 `html` 文件的 `<head>` 中。
我们尝试一下，通过在项目中添加一个新的 `style.css` 文件，并将其导入到我们的 `index.js` 中：
project
```
  webpack-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- style.css
    |- index.js
  |- /node_modules
```
src/style.css
```
.hello {
  color: red;
}
```
src/index.js
```
  import _ from 'lodash';
+ import './style.css';

  function component() {
    var element = document.createElement('div');

    // lodash 是由当前 script 脚本 import 导入进来的
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   element.classList.add('hello');

    return element;
  }

  document.body.appendChild(component());
```

构建
```
npm run build

> webpack-demo@1.0.0 build /Applications/XAMPP/xamppfiles/htdocs/webpack-study/AssetManagement/webpack-demo
> webpack

Hash: 4ec5a410cb136da1aba9
Version: webpack 4.8.3
Time: 4042ms
Built at: 2018-05-23 16:24:32
    Asset      Size  Chunks             Chunk Names
bundle.js  75.7 KiB       0  [emitted]  main
Entrypoint main = bundle.js
[4] ./node_modules/_css-loader@0.28.11@css-loader!./src/style.css 205 bytes {0} [built]
[5] ./src/style.css 1.13 KiB {0} [built]
[6] (webpack)/buildin/module.js 519 bytes {0} [built]
[7] (webpack)/buildin/global.js 509 bytes {0} [built]
[8] ./src/index.js 303 bytes {0} [built]
    + 4 hidden modules

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
再次在浏览器中打开 `index.html`，你应该看到 `Hello webpack` 现在的样式是红色。要查看 `webpack` 做了什么，请检查页面（不要查看页面源代码，因为它不会显示结果），并查看页面的 `head` 标签。它应该包含我们在 `index.js` 中导入的 `style` 块元素。

请注意，在多数情况下，你也可以进行 `CSS` 分离，以便在生产环境中节省加载时间。最重要的是，现有的 `loader` 可以支持任何你可以想到的 `CSS` 处理器风格 - `postcss`, `sass` 和 `less` 等。

### 加载图片
假想，现在我们正在下载 `CSS`，但是我们的背景和图标这些图片，要如何处理呢？使用 `file-loader`，我们可以轻松地将这些内容混合到 `CSS` 中：
```
sudo cnpm install --save-dev file-loader
```

webpack.config.js
```
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
+       {
+         test: /\.(png|svg|jpg|gif)$/,
+         use: [
+           'file-loader'
+         ]
+       }
      ]
    }
  };
```
现在，当你 `import MyImage from './my-image.png`'，该图像将被处理并添加到 `output` 目录，并且 `MyImage` 变量将包含该图像在处理后的最终 `url`。当使用 `css-loader` 时，如上所示，你的 `CSS` 中的 `url('./my-image.png')` 会使用类似的过程去处理。`loader` 会识别这是一个本地文件，并将 `'./my-image.png'` 路径，替换为输出目录中图像的最终路径。`html-loader` 以相同的方式处理 `<img src="./my-image.png" />`。
我们向项目添加一个图像，然后看它是如何工作的，你可以使用任何你喜欢的图像：
project
```
  webpack-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

src/index.js
```  import _ from 'lodash';
  import './style.css';
+ import Icon from './icon.png';

  function component() {
    var element = document.createElement('div');

    // Lodash，现在由此脚本导入
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
    element.classList.add('hello');

+   // 将图像添加到我们现有的 div。
+   var myIcon = new Image();
+   myIcon.src = Icon;
+
+   element.appendChild(myIcon);

    return element;
  }

  document.body.appendChild(component());
```

src/style.css
```
  .hello {
    color: red;
+   background: url('./icon.png');
  }
```
重新构建
```npm run build

> webpack-demo@1.0.0 build /Applications/XAMPP/xamppfiles/htdocs/webpack-study/AssetManagement/webpack-demo
> webpack

Hash: 97f460bdb8619f9cde24
Version: webpack 4.8.3
Time: 3943ms
Built at: 2018-05-23 16:35:08
                               Asset      Size  Chunks             Chunk Names
de92123ff58d303234f0b48ef1a5448f.png  50.2 KiB          [emitted]  
                           bundle.js  76.1 KiB       0  [emitted]  main
Entrypoint main = bundle.js
 [0] ./src/icon.png 82 bytes {0} [built]
 [6] ./node_modules/_css-loader@0.28.11@css-loader!./src/style.css 364 bytes {0} [built]
 [7] ./src/style.css 1.13 KiB {0} [built]
 [8] (webpack)/buildin/module.js 519 bytes {0} [built]
 [9] (webpack)/buildin/global.js 509 bytes {0} [built]
[10] ./src/index.js 435 bytes {0} [built]
    + 5 hidden modules

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
如果一切顺利，和 `Hello webpack` 文本旁边的 `img` 元素一样，现在看到的图标是重复的背景图片。如果你检查此元素，你将看到实际的文件名已更改为像 `de92123ff58d303234f0b48ef1a5448f.png` 一样。这意味着 `webpack` 在 `src` 文件夹中找到我们的文件，并成功处理过它！

###加载字体
那么，像字体这样的其他资源如何处理呢？`file-loader` 和 `url-loader` 可以接收并加载任何文件，然后将其输出到构建目录。这就是说，我们可以将它们用于任何类型的文件，包括字体。让我们更新 `webpack.config.js` 来处理字体文件：
```
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
        {
          test: /\.(png|svg|jpg|gif)$/,
          use: [
            'file-loader'
          ]
        },
+       {
+         test: /\.(woff|woff2|eot|ttf|otf)$/,
+         use: [
+           'file-loader'
+         ]
+       }
      ]
    }
  };
```

在项目中添加一些字体文件：
```
  webpack-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- 华康海报体简.TTC
    |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```
通过配置好 `loader` 并将字体文件放在合适的地方，你可以通过一个 `@font-face` 声明引入。本地的 `url(...)`指令会被 `webpack` 获取处理，就像它处理图片资源一样：

src/style.css
```+ @font-face {
+   font-family: 'MyFont';
+   src:  url('./华康海报体简.TTC') format('TTC'),
+   font-weight: 600;
+   font-style: normal;
+ }

  .hello {
    color: red;
+   font-family: 'MyFont';
    background: url('./icon.png');
  }
```

重新构建（貌似没有成功....不管了...

###加载数据
此外，可以加载的有用资源还有数据，如 `JSON` 文件，`CSV`、`TSV` 和 `XML`。类似于 `NodeJS`，`JSON` 支持实际上是内置的，也就是说 `import Data from './data.json'` 默认将正常运行。要导入 `CSV`、`TSV` 和 `XML`，你可以使用 `csv-loader` 和 `xml-loader`。让我们处理这三类文件：
```
sudo cnpm install --save-dev csv-loader xml-loader
```
webpack.config.js
```
const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
        {
          test: /\.(png|svg|jpg|gif)$/,
          use: [
            'file-loader'
          ]
        },
        {
          test: /\.(woff|woff2|eot|ttf|otf)$/,
          use: [
            'file-loader'
          ]
        },
+       {
+         test: /\.(csv|tsv)$/,
+         use: [
+           'csv-loader'
+         ]
+       },
+       {
+         test: /\.xml$/,
+         use: [
+           'xml-loader'
+         ]
+       }
      ]
    }
  };
```

给项目添加一些数据文件：
project
```
  webpack-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- data.xml
    |- my-font.woff
    |- my-font.woff2
    |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```
src/data.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Mary</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Call Cindy on Tuesday</body>
</note>
```
现在，可以 `import` 这四种类型的数据(`JSON`, `CSV`, `TSV`, `XML`)中的任何一种，所导入的 `Data` 变量将包含可直接使用的已解析 `JSON`：
src/index.js
```
  import _ from 'lodash';
  import './style.css';
  import Icon from './icon.png';
+ import Data from './data.xml';

  function component() {
    var element = document.createElement('div');

    // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
    element.classList.add('hello');

    // Add the image to our existing div.
    var myIcon = new Image();
    myIcon.src = Icon;

    element.appendChild(myIcon);

+   console.log(Data);

    return element;
  }

  document.body.appendChild(component());
```

构建
```
npm run build

> webpack-demo@1.0.0 build /Applications/XAMPP/xamppfiles/htdocs/webpack-study/AssetManagement/webpack-demo
> webpack

Hash: 5a2e0782bda56055905e
Version: webpack 4.8.3
Time: 4301ms
Built at: 2018-05-23 17:12:48
                               Asset      Size  Chunks                    Chunk Names
de92123ff58d303234f0b48ef1a5448f.png  50.2 KiB          [emitted]         
82bf13bac6520e21a3eabe7ebb07a2f3.TTC  2.49 MiB          [emitted]  [big]  
                           bundle.js  76.5 KiB       0  [emitted]         main
Entrypoint main = bundle.js
 [0] ./src/icon.png 82 bytes {0} [built]
 [1] ./src/data.xml 113 bytes {0} [built]
 [5] ./src/华康海报体简.TTC 82 bytes {0} [built]
 [8] ./node_modules/_css-loader@0.28.11@css-loader!./src/style.css 558 bytes {0} [built]
 [9] ./src/style.css 1.13 KiB {0} [built]
[10] (webpack)/buildin/module.js 519 bytes {0} [built]
[11] (webpack)/buildin/global.js 509 bytes {0} [built]
[12] ./src/index.js 487 bytes {0} [built]
    + 5 hidden modules

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/

WARNING in asset size limit: The following asset(s) exceed the recommended size limit (244 KiB).
This can impact web performance.
Assets: 
  82bf13bac6520e21a3eabe7ebb07a2f3.TTC (2.49 MiB)

WARNING in webpack performance recommendations: 
You can limit the size of your bundles by using import() or require.ensure to lazy load some parts of your application.
For more info visit https://webpack.js.org/guides/code-splitting/
```
控制台能看见打印的数据

在使用 `d3` 等工具来实现某些数据可视化时，预加载数据会非常有用。我们可以不用再发送 `ajax` 请求，然后于运行时解析数据，而是在构建过程中将其提前载入并打包到模块中，以便浏览器加载模块后，可以立即从模块中解析数据。

###全局资源
上述所有内容中最出色之处是，以这种方式加载资源，你可以以更直观的方式将模块和资源组合在一起。无需依赖于含有全部资源的 `/assets` 目录，而是将资源与代码组合在一起。例如，类似这样的结构会非常有用：
```
- |- /assets
+ |– /components
+ |  |– /my-component
+ |  |  |– index.jsx
+ |  |  |– index.css
+ |  |  |– icon.svg
+ |  |  |– img.png
```
这种配置方式会使你的代码更具备可移植性，因为现有的统一放置的方式会造成所有资源紧密耦合在一起。假如你想在另一个项目中使用 `/my-component`，只需将其复制或移动到 `/components` 目录下。只要你已经安装了任何扩展依赖(external dependencies)，并且你已经在配置中定义过相同的 `loader`，那么项目应该能够良好运行。
但是，假如你无法使用新的开发方式，只能被固定于旧有开发方式，或者你有一些在多个组件（视图、模板、模块等）之间共享的资源。你仍然可以将这些资源存储在公共目录(base directory)中，甚至配合使用 `alias` 来使它们更方便 `import` 导入。

