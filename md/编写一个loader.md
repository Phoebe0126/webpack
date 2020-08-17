### 编写自己的loader
loader的作用
简而言之，loader就是用来将一段代码转换成另一段代码的webpack插件。loader用于对模块的源代码进行转换，可以理解为webpack的编译器。它使得webpack可以处理一些非JavaScript文件，比如png、csv、xml、css、json等各种类型的文件，使用合适的loader可以让JavaScript的import导入非JavaScript模块。配置里的module.rules数组配置了一组规则，告诉webpack在遇到哪些文件时使用哪些loaders去加载和转换。

loader的原理
loader本质就是接收字符串(或者buffer)，再返回处理完的字符串(或者buffer)的过程。webpack会将加载的资源作为参数传入loader方法，交于loader处理，再返回。

loader主要概念
简单的loader格式
代码块
// 这是一个简单的loader
module.exports = function (source) {
    return source;
}
获取用户传入的options
代码块
const loaderUtils = require('loader-utils')
​
module.exports = function (source) {
    // 使用该方法获取options
    const options = loaderUtils.getOptions(this)
    return;
}
用户传入的options写在webpack.config.js里：

代码块
 rules: [{
            test: /\.js$/,
            use: [{
                loader: 'replaceLoader',
                // 用户传入的options
                options: {
                    name: 'zzs'
                }
            }]
        }]
返回其他资源
使用this.callback之后，相当于return，故return可以不用返回source

代码块
this.callback(
    // 当无法转换原内容时，给 Webpack 返回一个 Error
    err: Error | null,
    // 原内容转换后的内容
    content: string | Buffer,
    // 用于把转换后的内容得出原内容的 Source Map，方便调试
    sourceMap?: SourceMap,
    // 如果本次转换为原内容生成了 AST 语法树，可以把这个 AST 返回，
    // 以方便之后需要 AST 的 Loader 复用该 AST，以避免重复生成 AST，提升性能
    abstractSyntaxTree?: AST
);
​
同步和异步
loader转换流程可以是同步的，转换完成后再返回结果。

但在有些场景下转换的步骤只能是异步完成的，例如你需要通过网络请求才能得出结果，如果采用同步的方式网络请求就会阻塞整个构建，导致构建非常缓慢。

在转换步骤是异步时，你可以这样：

代码块
module.exports = function(source) { 
  // 告诉 Webpack 本次转换是异步的，Loader 会在 callback 中回调结果 
  var callback = this.async(); 
  someAsyncOperation(source, function(err, result, sourceMaps, ast) { 
      // 通过 callback 返回异步执行后的结果 
     callback(err, result, sourceMaps, ast); 
  }); 
}
ResolveLoader
代码块
module.exports = {
  resolveLoader:{
    // Webpack 会先去 node_modules 项目下寻找 Loader，如果找不到，会再去 ./loaders/ 目录下寻找。
    modules: ['node_modules','./loaders/'],
  }
}
编写一个简单的loader
loaders/replaceLoader.js

代码块
const loaderUtils = require('loader-utils')
​
module.exports = function (source) {
    const options = loaderUtils.getOptions(this)
    // 替换文件中的webpack为用户传入的options.name
    const result = source.replace('webpack', options.name)
    this.callback(null, result)
    return;
}
具体的webbpack.config.js如下：

代码块
const path = require('path')
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
    module: {
        rules: [{
            test: /\.js$/,
            use: [{
                // 自己写的loader名字
                loader: 'replaceLoader',
                options: {
                    name: 'zzs'
                }
            }]
        }]
    }
}
