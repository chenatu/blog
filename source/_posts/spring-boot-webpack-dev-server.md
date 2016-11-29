title: spring-boot 和 webpack-dev-server联合开发
date: 2016/9/28
categories:
- coding
tags:
- java
- spring
- javascript
- webpack
---
当前前后端架构分离的模式比较流行，前端用Nodejs或者ngnix等方式发布与渲染网页，后端程序只提供restful的数据接口。但对于一些小项目来说，并不想让前后端如此分离，还是希望用spring-boot的内置tomcat来serve static content。

如果只是用前端工具的话，webpack是一个很好的打包方式，webpack-dev-server给我们提供了很好的在线调试与修改。但是与spring-boot结合起来就不太协调。这时候就可以用到webpack-dev-server的代理模式了。通过webpack-dev-server来代理spring-boot中tomcat的端口（默认8080）

这里贴出我的一个配置文件

```
// webpack.config.js

var path = require('path');
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    devtool: "source-map",
    entry: [
        "webpack-dev-server/client?http://localhost:3000",
        "webpack/hot/only-dev-server",
        "./src/main/web/index.js"
    ],
    output: {
        path: "./src/main/resources/static",
        filename: "index.js",
        publicPath: 'http://localhost:3000/'
    },
    module: {
        loaders: [
            {test: /\.css$/, loader: "style!css"},
            {
                test: /\.js$/, loader: "babel-loader",
                exclude: /node_modules/,
                query: {
                    presets: ['es2015']
                }
            },
            { test: /\.(png|jpg|jpeg|gif|woff)$/, loader: 'url-loader?limit=8192' },
            { test: /\.html$/, loader: 'html'},
        ]
    },
    plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoErrorsPlugin(),
        new HtmlWebpackPlugin({
        template: './src/main/web/index.tmpl'
    })],
    devServer: {
        port: 3000,
        proxy: {
            '**': {
                target: 'http://localhost:8080',
                secure: false,
                prependPath: false
            }
        },
        publicPath: 'http://localhost:3000/',
        historyApiFallback: true
    }
};
```

在这里我们可以看到，通过webpack-dev-server的3000端口去代理8080端口。在package.json中添加

```
  "scripts": {
    "webpack": "webpack",
    "watch": "webpack-dev-server --inline"
  },
```

之后直接启动spring boot程序，然后`npm run watch`就可以通过访问3000端口来进行前端的热开发了