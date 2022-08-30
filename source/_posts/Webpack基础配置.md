---
title: Webpack基础配置
---

一些Webpack4的基础配置
<!-- more -->

代码库地址：https://github.com/blue-squirrel/webpack-learn

## 初始化项目

```
1. mkdir webpack-learn && cd webpack-learn
2. npm init
```

添加入口文件/src/index.js和webpack配置文件webpack.config.js

再安装webpack

```
npm install webpack webpack-cli -D
```

在script中添加打包命令

```
"build": "webpack --config 配置地址/webpack.config.js"
```

## 基础配置

### 配置html模板

执行打包命令只是将js文件打包了，但不能每次手动引入打包后的js

**html-webpack-plugin**

```
npm i -D html-webpack-plugin
```

```
// 自动插入生成文件到html中
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins:[
        new HtmlWebpackPlugin({
          template:path.resolve(__dirname,'../public/index.html')
        })
     ]
```

**clean-webpack-plugin**

```
npm install clean-webpack-plugin -D
```

```
// 清空上次打包文件
const {CleanWebpackPlugin} = require('clean-webpack-plugin')

plugins:[
        new CleanWebpackPlugin(),
    ]
```

### CSS支持

**为css添加浏览器前缀**

```
npm i -D postcss-loader autoprefixer  
```

```
rules:[
          {
            test:/\.css$/,
            use:['style-loader','css-loader'] // 从右向左解析原则
          },
          {
            test:/\.less$/,
            use:[MiniCssExtractPlugin.loader,'css-loader','less-loader',{
                loader: 'postcss-loader', // 为css添加浏览器前缀
                options: {
                    postcssOptions: {
                        plugins: [
                          ["autoprefixer"],
                        ],
                      }
                }
            }] // 从右向左解析原则
          },
```

**抽离CSS，将css额外打包出来**

```
npm i -D mini-css-extract-plugin
```

```
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  // 省略...
  module: {
    rules: [
      // 省略...
      {
        test: /\.(le|c)ss$/,
        exclude: /node_modules/,
        use: [
          MiniCssExtractPlugin.loader,
          // 省略...
        ]
      },
    ]
  },
  plugins: [
    // 省略...
    new MiniCssExtractPlugin({
      filename: 'css/[name].css',
      
    }),
  ],
}
```

### 实际项目开发相关

**实时更新并预览效果**

```
npm install webpack-dev-server -D
```

```
// 省略 ...
module.exports = {
  // 省略 ...
  devServer: {
    port: '3001', // 默认是 8080
    hot: true, // 热更新
    stats: 'errors-only', // 终端仅打印 error
    compress: true, // 是否启用 gzip 压缩
    proxy: {
      '/api': {
        target: 'http://0.0.0.0:80',
        pathRewrite: {
          '/api': '',
        },
      },
    },
  },
}
```

在package.json中添加命令

```
"dev": "webpack-dev-server --config 配置地址/webpack.dev.js --open",
```

**sourcemap配置**

项目报错的话，直接看编译后的代码，定位不精确。

开发环境中需要映射源代码

```
// 省略 ...
module.exports = {
  // 省略 ...
  devtool: 'cheap-module-source-map',
}
```

生产环境隐藏即可

```
devtool:'hidden-source-map',
```

**拆分环境**

不同的环境需要不同的打包配置

新建build目录，新建webpack.base.js、webpack.dev.js、webpack.prod.js

安装webpack配置合并组件

```
npm install webpack-merge -D
```

webpack.dev.js

```
const webpackConfig = require('./webpack.config.js')
const {merge} = require('webpack-merge')
module.exports = merge(webpackConfig,{
  mode:'development',
  devtool: 'cheap-module-source-map',
  devServer:{
    port:3000,
    hot:true,
    static:'../dist'
  },
  plugins:[
  ]
})
```

webpack.prod.js

```
const { merge } = require('webpack-merge');
const baseConfig = require('./webpack.config.js');

module.exports = merge(baseConfig, {
  mode: 'production',
  devtool: 'hidden-source-map',
});
```

**复制静态资源到打包目录**

有时第三方js插件没提供npm包，只有cdn地址或者文件需要自己下载。下载后放到public/js目录下，然后public/index.html文件中直接用script标签引入，但是本地调试或者打包后都是找不到这个文件的

public/index.html

```
<body>
    <div id="root"></div>
    <script src="./js/test.js"></script>
</body>
```

安装copy-webpack-plugin插件，构建时将public/js下静态资源复制到dist目录下

```
npm install copy-webpack-plugin -D
```

```
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  // 省略...
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        {
          from: '*.js',
          context: path.resolve(__dirname, "public/js"),
          to: path.resolve(__dirname, 'dist/js'),
        },
      ],
    })
```

