# react-scaffold

>搭建日志

## 写在前面

> 开始时间`2018-09-04`
本日志仅为[FireLeaf-React-Scaffold 2.x](https://github.com/NARUTOne/FireLeaf-React-Scaffold)搭建过程.
`node`下载最新版

**部分技术选择**：

- React
- React-router
- Redux
- axios
- webpack ^4.17.2

**UI**:

- antd
- less

**规范**：

- eslint
- stylelint

## init

```sh
# 创建项目
mkdir project-name && cd project-name

# init
npm init
```

创建项目需要文件夹

```sh
# build-tools
mkdir build

# products-config
mkdir config

# script
mkdir script

# main-src

mkdir src

# static

mkdir static

```

## webpack

> [https://www.webpackjs.com/](https://www.webpackjs.com/) 4.x

```sh
# npm install --save-dev webpack@<version>
npm install --save-dev webpack
npm install --save-dev webpack-cli
# merge
npm install --save-dev webpack-merge

```

webpack 配置`build/`

- webpack.base.config.js: 基础配置
- webpack.dev.config.js: dev模式配置
- webpack.prod.config.js: prod模式配置

## Babel

> Babel 是一个 JavaScript 编译器, 进行语法转换，可按需加载插件。

[babel 中文](https://babeljs.cn/)
[Babel 入门教程](http://www.ruanyifeng.com/blog/2016/01/babel.html)

### 开始

```sh
npm i babel-loader@7 babel-core --save-dev
```

- babel-loader: 这个包允许使用babel和webpack来转换JavaScript文件。
- babel-core: 如果某些代码需要调用Babel的API进行转码，就要使用babel-core模块。

```sh
npm install babel-preset-env babel-preset-stage-0 babel-preset-react --save-dev
```

- babel-preset-react 用于解析 JSX
- babel-preset-stage-0 用于解析 ES7 提案

- babel-preset-env: babel常用的转义器：相当于 es2015 ，es2016 ，es2017 及最新版本。
- stage-x:
  - Stage 0 - 稻草人: 只是一个想法，可能是 babel 插件。
  - Stage 1 - 提案: 初步尝试。
  - Stage 2 - 初稿: 完成初步规范。
  - Stage 3 - 候选: 完成规范和浏览器初步实现。
  - Stage 4 - 完成: 将被添加到下一年度发布。

```sh
npm install babel-plugin-transform-runtime --save-dev
```

- babel-plugin-transform-runtime: 类babel-polyfill, 按需polyfill

### .babelrc 配置

```json
{
  "presets": [
  ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }],
    "es2015",
    "react",
    "stage-0"
  ],
  "plugins": [
    "transform-runtime"
  ]
}
```

## 资源处理

> img、fonts、media

```sh
npm i url-loader file-loader --save-dev
```

[url-loader](https://www.npmjs.com/package/url-loader)
[file-loader](https://www.npmjs.com/package/file-loader)

## 编译css

```sh
npm install css-loader style-loader --save-dev
```

css-loader使你能够使用类似@import 和 url(...)的方法实现 require()的功能；

style-loader将所有的计算后的样式加入页面中； 二者组合在一起使你能够把样式表嵌入webpack打包后的JS文件中。

### 使用less

> 这里使用less, 其他预编译样式配置类似

```sh
npm i less less-loader --save-dev
```

### 样式兼容

```sh
npm i autoprefixer postcss-loader --save-dev
```

配置 `postcss.config.js`

```js
module.exports = {
  plugins: [
    require('autoprefixer')({
      browsers: [
        '>1%',
        'last 4 versions',
        'Firefox ESR',
        'not ie < 9', // React doesn't support IE8 anyway
      ],
      flexbox: 'no-2009',
    })
  ]
};
```

### 样式文件拆分

```sh
npm install --save-dev mini-css-extract-plugin
```

`webpack.base.config.js`配置

```js
{
  test: /\.css$/,
  exclude: /node_modules/,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    'postcss-loader'
  ]
},
{
  test: /\.less$/,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    'postcss-loader',
    'less-loader'
  ]
}
...
plugins: [
  new MiniCssExtractPlugin({
    // Options similar to the same options in webpackOptions.output
    // both options are optional
    filename: "[name].css",
    chunkFilename: "[id].css"
  })
]
```

## webpack-config

### webpack-dev

> dev模式下webpack配置

```sh
npm i --save-dev html-webpack-plugin open-browser-webpack-plugin
```

### webpack-prod

> prod模式下webpack配置

```sh
npm i --save-dev optimize-css-assets-webpack-plugin
```

### webpack-server

> webpack 开发下的 server配置, 主要有下面两种方式

#### webpack-dev-server

> [https://www.webpackjs.com/guides/development/#%E4%BD%BF%E7%94%A8-webpack-dev-server](https://www.webpackjs.com/guides/development/#%E4%BD%BF%E7%94%A8-webpack-dev-server)

```sh
npm i --save-dev webpack-dev-server
```

#### express + + webpack-dev-middleware

> express 服务（node）+ webpack-dev-middleware + webpack-hot-middleware

```sh
npm i --save-dev webpack-dev-middleware webpack-hot-middleware eventsource-polyfill express

# server log
npm i --save-dev rimraf

```

### webpack-build-prod

```sh
# 终端 spinner
npm i --save-dev ora rimraf chalk
```

### webpack 其他配置

1、copy静态资源 static

```sh
npm i --save-dev copy-webpack-plugin
```

2、压缩打包文件

```sh
npm i --save-dev zip-webpack-plugin
```

## 规范

> 代码、样式编码规范，代码简洁易读，提升项目开发效率。:blush:

- [http://eslint.cn/](http://eslint.cn/) : 代码规范
- [https://stylelint.cn/](https://stylelint.cn/) ： 样式规范
- [http://editorconfig.org/](http://editorconfig.org/) ： 编码规范

```sh
npm install babel-eslint eslint-plugin-react eslint stylelint stylelint-config-standard --save-dev
```

### vscode + eslint

> 自动检测排查，补全修复

```js
"eslint.autoFixOnSave": true,
    "eslint.validate": [
        // "javascript",
        // "javascriptreact",
        // "html",
        // "vue"
        {
            "language": "javascript",
            "autoFix": true
        },
        {
            "language": "javascriptreact",
            "autoFix": true
        },
        {
            "language": "vue",
            "autoFix": true
        },
        {
            "language": "jsx",
            "autoFix": true
        },
        {
            "language": "html",
            "autoFix": true
        }
    ],
```

### stylelint autofix

> 样式自动修复

```sh
npm install stylelint-webpack-plugin --save-dev
```

webpack-config

```js
new StyleLintPlugin({
  // 正则匹配想要lint监测的文件
  files: ['src/**/*.l?(e|c)ss'],
  cache: true,
  fix: true
})
```

## running

> package.json 配置 script命令

```js

"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "lint": "eslint --ext .js src script config test && npm run lint:style",
  "lint:fix": "eslint --fix --ext .js src script config test && npm run lint:style",
  "lint-staged": "lint-staged",
  "lint-staged:js": "eslint --ext .js",
  "lint:style": "stylelint \"src/**/*.less\" --syntax less",
  "start": "node script/server.js", // express server
  "dev": "node script/dev-server.js", // webpack dev  server
  "build": "node script/prod.js" // webpack build prod
}

```

> ☝ 注意 写进package.json中不能带有注释

## 持续集成服务 Travis CI

> 绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。

- [持续集成服务 Travis CI 教程](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)
- [https://travis-ci.org/](https://travis-ci.org/)

## React

> 集成React

```sh
npm i --save react react-dom prop-types
```

### react-router

> 升级 4.x [https://zhuanlan.zhihu.com/p/27433116](https://zhuanlan.zhihu.com/p/27433116)

- [react-router](https://reacttraining.com/)
- [v3 升级 v4](https://blog.csdn.net/chenqiuge1984/article/details/80131519)
- [Dynamic Routing 思想](https://github.com/wayou/wayou.github.io/issues/16)
- [React Router v4 几乎误我一生](https://zhuanlan.zhihu.com/p/27433116)

```sh
npm i --save react-router-dom

# 导入 history
npm i --save history
```

**history跳转**:

[https://github.com/brickspert/blog/issues/3](https://github.com/brickspert/blog/issues/3)

### redux

> redux数据处理 [https://cn.redux.js.org/](https://cn.redux.js.org/)

```sh
npm i --save redux react-redux
```

**react-router-redux**
这个包的正式版4.x不支持react-router v4。你需要用 alpha 版 的react-router-redux。在package.json 里加入react-router-redux~5.0.0或者用yarn：

```sh
yarn add react-router-redux@5.0.0
```

### react应用热更新

[https://github.com/gaearon/react-hot-loader/issues/511#issuecomment-288673129](https://github.com/gaearon/react-hot-loader/issues/511#issuecomment-288673129)

```sh
npm i --save-dev react-hot-loader
```

```js
import React from 'react';
import { render, unmountComponentAtNode } from 'react-dom';
import { Provider } from 'react-redux';
import { AppContainer } from 'react-hot-loader';  
import thunk from 'redux-thunk';
// 引入路由配置模块
import RouterList from './router/';
import { createStore, applyMiddleware } from 'redux';
import reducer from './reducer/';

// redux 注入操作
const middleware = [thunk];
const store = createStore(reducer, applyMiddleware(...middleware));
// console.log(store.getState());

const mountNode = document.getElementById('app'); // 设置要挂在的点

const hotRender = Component => render(
  <AppContainer>
    <Provider store={store}>
      <Component />
    </Provider>
  </AppContainer>
, mountNode);

hotRender(RouterList);

if(process.env.NODE_ENV === 'development') {
  if(module.hot) {
    module.hot.accept('./router/', (err) => {
      if (err) {
        console.log(err);
      }
      const RouterList = require('./router/').default;
      unmountComponentAtNode(mountNode);
      hotRender(RouterList);
    });
  }
}

```

### 异步import, code split

> 异步导入组件，代码拆分

```sh
npm i --save-dev babel-plugin-syntax-dynamic-import

npm i --save react-loadable
```

配置使用
[解决异步loadable引入导致react-hot-loader失效](https://medium.com/@giang.nguyen.dev/hot-loader-with-react-loadable-c8f70c8ce1a6)

```js
{
  "presets": [
    "react"
  ],
  "plugins": [
    "syntax-dynamic-import"
  ]
}

import Loadable from 'react-loadable';
import Loading from './Loading';

const LoadableComponent = Loadable({
  loader: () => import('./Dashboard'),
  loading: Loading,
})

export default class LoadableDashboard extends React.Component {
  render() {
    return <LoadableComponent />;
  }
}
```

## Antd

> Ant Design 的 React 实现, 蚂蚁UI组件库 [ant-design](https://ant.design/docs/react/introduce-cn)

```sh
# install
npm i antd --save
# 按需加载
npm i babel-plugin-import --save-dev
```

**babel-config**:

```js
["import", { "libraryName": "antd", "libraryDirectory": "es", "style": true }]
```

### 定制主题

- [Errors when importing antd.less using less-loader #7850](https://github.com/ant-design/ant-design/issues/7850)
- [定制主题中单独使 webpack 进行 theme 定制的更改步骤补充 #8035](https://github.com/ant-design/ant-design/pull/8035/commits/7fef8e993a0049579d3a00de4691efef255127b6)

**config/theme.js**:

```js
/**
 * antd theme config
 */

const defaultColor = '#4285f4';

module.exports = () => {
  return {
    'primary-color': defaultColor,
    'link-color': defaultColor,
    'border-radius-base': '3px',
    'menu-collapsed-width': '70px',
  };
};

// const fs = require('fs')
// const path = require('path')
// const lessToJs = require('less-vars-to-js')

// module.exports = () => {
//   const themePath = path.join(__dirname, './src/utiles/style/theme.less')
//   return lessToJs(fs.readFileSync(themePath, 'utf8'))
// }

```

**package.json**:

```js
"theme": "./config/theme.js",
```

**webpack.base.config.js**:

```js
// 获取theme
const fs = require('fs');
const pkgPath = path.resolve(__dirname, './package.json');
const pkg = fs.existsSync(pkgPath) ? require(pkgPath) : {};
let theme = {};
if (pkg.theme && typeof pkg.theme === 'string') {
  let cfgPath = pkg.theme;
  if (cfgPath.charAt(0) === '.') {
    cfgPath = path.resolve(__dirname, cfgPath);
  }
  const getThemeConfig = require(cfgPath);
  theme = getThemeConfig();
} else if (pkg.theme && typeof pkg.theme === 'object') {
  theme = pkg.theme;
}

...

{
  test: /\.less$/,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    'postcss-loader',
    {
      loader: 'less-loader',
      options: {
        "sourceMap": true,
        "modules": false,
        "modifyVars": theme,
        'javascriptEnabled': true
      }
    }
  ]
}
```

## webpack 优化

参考： [webpack4.0打包优化策略](https://juejin.im/post/5abbc2ca5188257ddb0fae9b)