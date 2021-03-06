---
title: webpack 从入门到放弃
date: 2017-03-11 18:06:04
categories: JavaScript  
tags:
- webpack
- JavaScript
---

## 简介

Webpack + ES6 已经成为目前最流行的前端解决方案，本文是 Webpack2 学习教程。

在 「[What is webpack](https://webpack.github.io/docs/what-is-webpack.html)」一文中作者讲述了自己为什么要开发出 webpack。

1. 切分代码依赖树到不同的代码块，按需加载
2. 保持更少的初始化加载时间
3. 把任何静态资源都视为模块
4. 把任何第三方类库也当作模块
5. 在模块打包中每一部分都允许自定义
6. 更加适合大型项目

<!-- more -->
## 使用

### 安装

新建 webpack-demo 文件夹，安装 webpack 到 dev

``` shell
$ mkdir webpack-demo
$ npm init -y
$ npm i webpack --save-dev
```

### 命令行打包

新建一个 hello.js 文件

```javascript
function hello () {
    alert('Hello webpack')
}
hello()
```

在命令行中输入下面内容进行打包

```shell
$ webpack hello.js hello.bundle.js
```

打开打包后的文件发现里面注入了很多 webpack 所需的一些内置函数，比如 **`__webpack_require__`**，除此之外，webpack 还对我们写的代码进行编号，比如刚才我们写的 hello function 在 hello.bundle.js 中的编号就是  `/* 0 */`。

### 引入css文件

新建 style.css 文件

```css
body {
    background-color: gray;
}
```

在 hello.js 中引入该文件

```javascript
require('./style.css')
```

再次使用刚才的命令打包，发现命令行报错

```
ERROR in ./style.css
Module parse failed: D:\webpack-demo\style.css Unexpected token (1:11)
You may need an appropriate loader to handle this file type.
| html, body {
|       background-color: gray;
 @ ./hello.js 2:0-22
```

错误提示很明显：模块解析错误，你可能需要一个合适的 loader 去处理这种类型的文件。

webpack 默认不支持 css 文件类型，所以我们来安装 css-loader 和 style-loader

```shell
$ npm i css-loader style-loader --save-dev
```

css-loader 是使 webpack 可以处理 css 文件；style-loader 把 css-loader 处理完的代码，新建一个 style 标签，插入到 HTML 代码中。

然后将这两个 loader 引入 hello.js

```javascript
require('style-loader!css-loader!./style.css')
```

再次运行打包命令就可以在 hello.bundle.js 中找到下面这句话

```javascript
exports.push([module.i, "body {\r\n\tbackground-color: gray;\r\n}", ""]);
```

为了查看效果，我们新建 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webpack demo</title>
</head>
<body>
    <script src="./hello.bundle.js"></script>
</body>
</html>
```

打开 index.html 查看效果，发现 style-loader 在 head 中插入了一个 style 标签将 css 插入 html 中。

## webpack [配置文件](https://webpack.js.org/configuration/)

在命令行中输入 `webpack` 命令，webpack 会自动寻找 webpack.config.js 文件，并按照里面的配置对项目进行打包。还可以通过 `--config` 参数指定 webpack 配置文件。

webpack.config.js 使用 CommonJS 规范，下面是一个最基础的配置文件

```javascript
module.exports = {
  entry: './src/script/main.js',
  output: {
    path: './dist/js',
    filename: 'bundle.js'
  }
}
```

`entry` 参数表明我们的打包是从哪个文件开始的，`output` 参数定义打包后的文件如何存储。 

如果需要使用一些 webpack 的参数，可以使用 npm 脚本来实现，比如

```javascript
"scripts": {
    "webpack": "webpack --display-modules --sort-modules-by size"
}
```

上面是我们分析 webpack 打包后文件常用的方式，把每个 modules 显示出来，并且按照文件大小排序。


## webpack 几个重要概念

### [entry](https://webpack.js.org/concepts/entry-points/)

webpack 根据 entry 创建所有应用程序依赖图表，entry 告诉 webpack 从哪里开始，并遵循着依赖关系图打包。

entry 有以下几种写法

```javascript
entry: './src/app.js'
entry: ['./src/app.js', './src/vendors.js']
entry: {
    main: './src/app.js'
}
entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
}
entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
}
```

指定多入口主要为了解决两种场景，一个是将业务代码和框架代码分割，一个是为了处理多页面应用。使用 [CommonsChunkPlugin](https://github.com/webpack/docs/wiki/list-of-plugins#commonschunkplugin) 插件可以将公共的类库代码打包成一个 common 模块。这样在多页面程序中可以把共用代码缓存起来，方便其他页面使用。

### [output](https://webpack.js.org/concepts/output/)

output 参数告诉 webpack 如何把编译后的文件写入到磁盘里，无论有多少个 entry 都只有一个 output 配置。一般形式的写法如下：

```javascript
output: {
    filename: 'bundle.js',
    path: __dirname + '/build'
}
```
output.path 是一个绝对路径，filename 指生产打包文件后的名称

假如 entry 为多入口，使用上述写法只会生产一个 bundle.js，不符合我们代码分割的需求，那么我们可以用一些占位符来表示输出的结果。一共有四种占位符：[id], [name], [hash], [chunkhash]。注意 [hash] 指的是本次打包的 hash，这个 hash 在 webpack 打包时日志的第一行显示。而 [chunkhash] 是每一个 chunk 自己的 hash 值。

```javascript
entry: {
    app: './src/app.js',
    search: './src/search.js'
},
output: {
    filename: '[name]-[chunkhash].js',
    path: __dirname + '/build'
}
```

hash 值由 md5 算法生成，可以当做每个文件的版本号，这点对于我们管理产品时每次只上线被更改的文件非常有用。如果觉得默认 hash 值太长了，可以通过 [chunkhash:8] 来指定 hash 位数。

通常我们上线产品会使用 cdn 加速静态资源文件的获取，我们可以把 cdn 写入到 output.publicPath 中。publicPath 表示如果产品上线，js 的路径就会自动加上 publicPath。

```javascript
output: {
    filename: '[name]-[chunkhash].js',
    path: __dirname + '/build',
    publicPath: 'http://cdn.example.com/'
}
```

### [loaders](https://webpack.js.org/concepts/loaders/)

webpack 中把所有的资源都当做一个模块，无论这个文件是代码文件，还是图片文件，只要有对应的 loader 均可以在 webpack 中转换使用，这也是 webpack 最大的优势所在。

前面「引入css文件」中已经展示了如何使用 loader，通常配置方式如下：

```javascript
module: {
    rules: [
        {
            test: /\.(js|jsx)$/,
            use: 'babel-loader'
        }
    ]
}    
```
test 说明了当前 loader 能处理那些类型的文件的正则匹配，use 则指定了 loader 的类型。

注：这里说一下 webpack1 与 webpack2 的区别，在 webpack1 中，使用 module.loaders 声明 loader，而 webpack2 中使用功能更为强大的 module.rules。 为了兼容旧版，module.loaders 语法仍然有效，旧的属性名依然可以被解析。

loader 还可以在使用的时候传入相关的参数，比如我们使用 css-loader 时

```javascript
module: {
    rules: [{
        test: /\.css$/,
        use: [
            'style-loader', 
            {
                loader: 'css-loader',
                options: {
                    importLoaders: 1
                }
            }
        ]
    }]
}
```

注：在 webpack 1 中，loader 可以链式调用，上一个 loader 的输出被作为输入传给下一个 loader，通常被用 ! 连写，如 `loader: "style-loader!css-loader!less-loader"`。这一写法在 webpack 2 中只在使用旧的选项 module.loaders 时才有效。使用 rule.use 配置选项，use 可以设置为一个 loader 数组。使用 module.rules 时，如果只有一个 loader，既可以用 loader 又可以用 use，但是如果是多 loader，则只能使用 use。

#### 处理 ES6 语法

首先安装 babel

```shell
$ npm install babel-loader babel-core --save-dev
```

修改 webpack 配置文件

```javascript
module: {
    rules: [{
        test: /\.js$/,
        exclude: path.resolve(__dirname, 'node_modules/'),
        include: path.resolve(__dirname, 'src/'),
        loader: "babel-loader"
    }]
}
```
注意这里一定要加上 exclude 或者 include，因为 babel-loader 处理的速度非常慢。
然后还需要指定所用 ECMAScript 的版本，假如使用 ES6 语法

```shell
$ npm install babel-preset-es2015 --save-dev
```

告诉 webpack babel 使用哪个版本的 preset 有三种方式

(1) 在 webpack 中声明

```javascript
module: {
    rules: [{
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader",
        query: {
            presets: ["es2015"]
        }
    }]
}
```

注： 如果 loader 需要传参数的话，既可以写成 query 的形式，也可以写成像 url 传参一样的形式：

```javascript
rules: [{
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "babel-loader?presets=es2015"
}]
```

但是如果为多 loader 的话，只能用 use + options 的形式。

(2) 在根目录创建 .babelrc 文件，文件内容如下

```
{
  "presets": ["es2015"]
}
```

(3) 在 package.json 中指定 preset

```javascript
"babel": {
  "presets": ["es2015"]
}
```


### [plugins](https://webpack.js.org/concepts/plugins/)

插件是 wepback 的支柱功能。在你使用 webpack 配置时，webpack 自身也构建于同样的插件系统上！插件目的在于解决 loader 无法实现的其他事，在[这个页面](https://webpack.js.org/plugins/)你可以看到一些 webpack 常用的插件。

由于 plugin 可以传递参数，你必须在 wepback 配置中，向 plugins 属性传入 new 实例。

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); //通过 npm 安装
const webpack = require('webpack'); //访问内置的插件
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    loaders: [
      {
        test: /\.(js|jsx)$/,
        loader: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

### html-webpack-plugin 使用

下面我们以最常用的 **[html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)** 为例，讲解插件的用法。

首先使用 npm 安装插件

```shell
$ npm i html-webpack-plugin --save-dev
```

然后在 webpack.config.js 配置文件中使用。

```javascript
const htmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    entry: {
        app: './src/script/main.js'
    },
    output: {
        filename: '[name]-[chunkhash:8].js',
        path: __dirname + '/build'
    },
    plugins: [
        new htmlWebpackPlugin({
            filename: 'index-[hash].html',
            template: 'index.html'
        })
    ]
}
```

**html-webpack-plugin** 插件还能接受一些其它参数，比如`title`、`inject: (true | 'head' | 'body' | false)`、`favicon`、`minify`、`hash`、`cache`等。

还可以设置一些自定义的参数，在 html 文件中通过类似 js 模板语言的方式进行引用。
比如在 webpack 配置文件中

```javascript
plugins: [
    new htmlWebpackPlugin({
        template: 'index.html',
        date: new Date()
    })
]
```

然后在 index.html 中使用 `<%= htmlWebpackPlugin.options.date %>` 对 date 进行引用，这样就给了我们更大的自由度，用相同的 html 模板生成不同的 html 文件。

通过加上 minify 来实现对 html 文件的压缩，minify 传入一个 [html-minify](https://github.com/kangax/html-minifier#options-quick-reference) 对象。

```javascript
plugins: [
    new htmlWebpackPlugin({
        template: 'index.html',
        minify: {
            removeComments: true,
            collapseInlineTagWhitespace: true,
            collapseWhitespace: true
        }
    })
]
```

对于一个多页面应用程序，需要生成多少个页面，就 new 多少个 htmlWebpackPlugin 实例。假如不同的页面依赖不同的 chunks， 那么我们可以使用 chunks 参数指定当前页面所使用的 chunks。也可以使用 excludeChunks 来指定排除了某些 chunks 以后的全部 chunks。 

```javascript
const htmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    entry: {
        a: './src/script/a.js',
        b: './src/script/b.js',
        c: './src/script/c.js'
    },
    output: {
        filename: 'js/[name]-[chunkhash:8].js',
        path: __dirname + '/build'
    },
    plugins: [
        new htmlWebpackPlugin({
            filename: 'a.html',
            template: 'index.html',
            title: 'page a',
            chunks: ['a']
        }),
        new htmlWebpackPlugin({
            filename: 'b.html',
            template: 'index.html',
            title: 'page b',
            chunks: ['b']
        }),
        new htmlWebpackPlugin({
            filename: 'c.html',
            template: 'index.html',
            title: 'page c',
            chunks: ['c']
        })
    ]
}
```

对应的 html 模板文件为：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
</body>
</html>
```

运行`npm run webpack`后生成 3 个 html 文件，分别引入其所需要的依赖。

## webpack 处理资源文件

### 样式文件 less/sass

假如有这么一段 less

```css
.layer {
    width: 600px;
    height: 200px;
    background-color: green;
    > div {
        width: 400px;
        height: 200px;
        background: gray;
    }
}
```

首先安装 less 和 less-loader

```shell
$ npm install less less-loader --save-dev
```

在 webpack 配置文件中加入 less-loader

```javascript
module: {
    rules: [
    {
        test: /\.less$/,
        use: [
            "style-loader",
            "css-loader",
            "less-loader"
       ]
    }]
},
```

loader 的执行顺序为从后往前执行，所以其顺序为 less-loader -> css-loader -> style-loader。 如果需要引入 postcss-loader 的话，应该放在 less-loader 和 css-loader 中间。

###  图片文件

图片文件一般使用 [file-loader](https://github.com/webpack-contrib/file-loader) 配合 [url-loader](https://github.com/webpack-contrib/url-loader)，如果有压缩需求的话，可以使用 [image-webpack-loader](https://github.com/tcoopman/image-webpack-loader)

安装两个 loader

```shell
$ npm i file-loader url-loader --save-dev
```

url-loader 的功能与 file-loader 十分相似，不同的是 url-loader 可以指定一个 limit 参数， 当图片或者文件的大小大于 limit 的时候，url-loader 把资源直接交给 file-loader 处理，而当资源小于 limit 的时候，url-loader 会把图片转为 base64 的编码，并直接打包到引用的文件中。

file-loader 打包的文件通过 http 请求获取，url-loader 打包的文件通过 base64 的方式获取，这两个方法各有各的优势。通过 http 载入的图片可以享受到浏览器的图片缓存，当图片重复使用次数比较多的时候具有一定的便利。base64 的方式引入图片可以降低 http 请求的次数，但是也会带来一定程度的代码冗余。

(1) 使用 file-loader

```javascript
module: {
    rules: [{
        test: /\.(ico|jpe?g|png|svg|gif)$/i,
        loader: 'file-loader'
    }]
}
```

(2) 使用 url-loader

```javascript
module: {
    rules: [{
        test: /\.(ico|jpe?g|png|svg|gif)$/i,
        loader: 'url-loader',
        query: {
            limit: 10000,
            name: 'assets/[name]-[hash].[ext]'
        }
    }]
}
```

(3) image-webpack-loader 可以对图片文件进行压缩，并且配合 url-loader 和 file-loader 共同使用

```javascript
module: {
    rules: [{
        test: /\.(ico|jpe?g|png|svg|gif)$/i,
        use: [
            'url-loader?limit=10000&name=assets/[name]-[hash].[ext]',
            'image-webpack-loader?options={}'
        ]
    }]
}
```

loader 的参数也可以通过 options 传递

```javascript
{
test: /\.(ico|jpe?g|png|svg|gif)$/i,
use: [
    {
        loader: 'url-loader',
        options: {
            limit: 10000,
            name: 'assets/[name]-[hash].[ext]'
        }
    },
    {
        loader: 'image-webpack-loader',
        options: {}
    }]
}
```

image-webpack-loader 可以针对不同的图片类型就行压缩，详细的信息可以在[官网](https://github.com/tcoopman/image-webpack-loader)里面查询。

注：在 image-webpack-loader 实际使用过程中，必须传入一个 options 参数，否则会报错，使用的时候注意一下。

>ERROR in   Error: Child compilation failed:
  Module build failed: TypeError: Cannot read property 'bypassOnDebug' of null

## 总结

本文只是 webpack 打包的一些知识，只涉及到一些基本使用，关于 webpack 在项目中的实际应用，以及打包的一些技巧和优化，会在下一节中讲起。

