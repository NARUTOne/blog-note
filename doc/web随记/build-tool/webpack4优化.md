# webpack4 优化

> webpack4 打包优化。**下面代码均为示例，使用需要调整**

## loader配置

### 缩小文件匹配范围 + 缓存loader

> `include/exclude`, 排除node_modules下的文件 从而缩小了loader加载搜索范围 高概率命中文件；`cacheDirectory`是loader的一个特定的选项，默认值是false。指定的目录`(use: 'babel-loader?cacheDirectory=cacheLoader')`将用来缓存loader的执行结果，减少webpack构建时Babel重新编译过程。

```js
{
  test: /\.js$/,
  use: 'babel-loader?cacheDirectory', // 缓存loader执行结果
  include: [utils.resolve('node_modules/...'), utils.resolve('src')], // 多包含
  exclude: /node_modules/
}
```

设置了cacheDirectory，将使用默认的缓存目录`(node_modules/.cache/babel-loader)`，如果在任何根目录下都没有找到 node_modules 目录，将会降级回退到操作系统默认的临时文件目录。

## resolve配置

### resolve.modules

> 模块查找路径优化

在 js 里出现 `import 'vue'` 这样不是相对、也不是绝对路径的写法时，会去 `node_modules` 目录下找。但是默认的配置，会采用向上递归搜索的方式去寻找，但通常项目目录里只有一个 `node_modules` ，且是在项目根目录，为了减少搜索范围，可以直接写明 `node_modules` 的全路径

```js
{
  resolve: {
    modules: [
      utils.resolve('node_modules')
    ]
  }
}
```

## module.noParse

> 忽略编译模块, 注意使用编译好的依赖

```js
// 忽略对jquery lodash的进行递归解析
module: {
  // noParse: /jquery|lodash/

  // 从 webpack 3.0.0 开始
  noParse: function(content) {
    return /jquery|lodash/.test(content)
  }
}

```

## DLL动态链接

> 在一个动态链接库中可以包含其他模块调用的函数和数据，动态链接库只需被编译一次，在之后的构建过程中被动态链接库包含的模块将不会被重新编译,而是直接使用动态链接库中的代码。

- 将web应用依赖的基础模块抽离出来，打包到单独的动态链接库中。一个链接库可以包含多个模块。
- 当需要导入的模块存在于动态链接库，模块不会再次打包，而是去动态链接库中去获取。
- 页面依赖的所有动态链接库都需要被加载。

创建DLL

```js
// webpack.dll.config.js
module.exports = {
    entry: {
        react: ['react', 'react-dom']
    },
    output: {
        filename: '[name].dll.js', // 动态链接库输出的文件名称
        path: path.join(__dirname, 'dist'), // 动态链接库输出路径
        libraryTarget: 'var', // 链接库(react.dll.js)输出方式 默认'var'形式赋给变量 b
        library: '_dll_[name]_[hash]' // 全局变量名称 导出库将被以var的形式赋给这个全局变量 通过这个变量获取到里面模块
    },
    plugins: [
        new webpack.DllPlugin({
            // path 指定manifest文件的输出路径
            path: path.join(__dirname, 'dist', '[name].manifest.json'),
            name: '_dll_[name]_[hash]', // 和library 一致，输出的manifest.json中的name值
        })
    ]
}

```

配置

```js
plugins: [
    // 当我们需要使用动态链接库时 首先会找到manifest文件 得到name值记录的全局变量名称 然后找到动态链接库文件 进行加载
    new webpack.DllReferencePlugin({
       manifest: require('./dist/react.manifest.json')
    })
]
```

插入html

```npm
npm i html-webpack-plugin html-webpack-include-assets-plugin -D
```

```js
new webpack.DllReferencePlugin({
    manifest: require('./dist/react.manifest.json')
}),
new HtmlWebpackPlugin({
    template: path.join(__dirname, 'src/index.html')
}),
new HtmlIncludeAssetsPlugin({
    assets: ['./react.dll.js'], // 添加的资源相对html的路径
    append: false // false 在其他资源的之前添加 true 在其他资源之后添加
});
```

## 区分环境

> 区分构建环境, 设置不同环境下所需配置变量

`cross-env`模块设置环境变量

```npm
npm i cross-env -D
```

配置使用

```js
{
"start": "cross-env NODE_ENV=development node script/server.js",
"dev": "cross-env NODE_ENV=development node script/dev-server.js",
"dev-build": "cross-env NODE_ENV=dev node script/prod.js",
"pre-build": "cross-env NODE_ENV=pre node script/prod.js",
"build": "cross-env NODE_ENV=production node script/prod.js"
}
```
