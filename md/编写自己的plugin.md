#### plugin的作用
plugin用于扩展webpack的功能，包括打包、优化、压缩、搭建服务器等等与构建相关的事情。plugins配置项接受一个数组，数组里每一项都是一个要使用的plugin实例，plugin需要的参数通过构造函数传入。需要掌握的是plugin本身提供的配置项。clean-webpack-plugin（清除上一次打包的目录文件）、html-webpack-plugin（自动创建html引入打包好的js和css）

#### 原理
一个 webpack plugin 基本包含以下几步：

一个 JavaScript 函数或者类

在函数原型（prototype）中定义一个注入compiler对象的apply方法。

apply函数中通过compiler插入指定的事件钩子，在钩子回调中拿到compilation对象

使用compilation操纵修改webapack内部实例数据。

异步插件，数据处理完后使用callback回调

#### 解释
compiler
Compiler 对象包含了 Webpack 环境所有的的配置信息。包含 options，hook，loaders，plugins 这些信息，这个对象在 Webpack 启动时候被实例化，它是全局唯一的，可以简单地把它理解为 Webpack 实例。

##### 钩子：

entryOption：在 entry 配置项处理过之后，执行插件

emit：生成资源到 output 目录之前。

failed：编译(compilation)失败

compilation
Compilation 对象包含了当前的模块资源、编译生成资源、变化的文件等。当 Webpack 以开发模式运行时，每当检测到一个文件变化，一次新的 Compilation 将被创建。Compilation 对象也提供了很多事件回调供插件做扩展。通过 Compilation 也能读取到 Compiler 对象。

钩子：

buildModule：在模块构建开始之前触发。

optimize：优化阶段开始时触发。

beforeChunkAssets：在创建 chunk 资源(asset)之前

additionalAssets：为编译(compilation)创建附加资源(asset)

编写一个plugin
这个plugin的作用是把外部的js进行打包并引入到项目的index.html中。

步骤：

fs读取options传入的文件

使用compilation.assets处理打包到dist里的文件，将外部js文件打包，并在index.html的</body>标签前插入script标签，src为该js文件。

test.js文件就可以插入到index.html文件里了～

testPlugin.js文件：

代码块
const { compilation } = require("webpack");
const fs = require('fs');
​
class TestPlugin {
    constructor (options = {}) {
        this.options = options;
    }
    apply (compiler) {
        // 在打包之前完成这个引入js到html操作，故使用emit钩子
        compiler.hooks.emit.tapAsync('TestPlugin', (compilation, callback) => {
            // 读取用户传入到js文件
            let template = fs.readFileSync(this.options.template, 'UTF-8');
            // 使用compilation的assets处理资源文件
            // 把文件导入到dist文件夹
            compilation.assets[this.options.filename || 'test.js'] = {
                source: () => template,
                size: () => template.length
            };
            // 将js文件引入index.html中
            let source = compilation.assets['index.html'].source();
            // body标签插入
            source = source.replace(
                /<\/(.*?)>(.*?)<\/body>$/m,
                `</$1><script src="${this.options.filename ||
                  'test.js'}"></script></body>`,
            );
            compilation.assets['index.html'] = {
                source: () => source,
                size: () => source.length
            }
            callback();
        });
    }
}
​
module.exports = TestPlugin;
webpack.config.js文件：

代码块
const path = require('path')
// 引入
const TestPlugin = require('./plugins/testPlugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
​
module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].js'
    },
    resolveLoader: {
        modules: ['node_modules', './loaders']
    },
    plugins: [
        // 记得提前npm install它喔～
        new HtmlWebpackPlugin(),
        // 使用
        new TestPlugin({
            filename: 'test.js',
            template: path.resolve(__dirname, './src/common/test.js')
        })
    ],
    module: {
        rules: [{
            test: /\.js$/,
            use: [{
                loader: 'replaceLoader',
                options: {
                    name: 'zzs'
                }
            }]
        }]
    }
}
