# TypeScript-React-Template

> TS结合React项目模板搭建

- [Creact React App](https://create-react-app.dev/docs/getting-started)
- [TypeScript](https://www.typescriptlang.org/)
- [图雀 类型即正义：TypeScript 从入门到实践系列](https://juejin.im/post/6844904167840940039#heading-5)

## 创建

初始一个 React 应用的最佳方式那么一定是 React 官方维护的 Create React App 脚手架了，我们打开终端，运行如下命令来初始化一个 TypeScript 版本的 React 应用：

```bash
$ npx create-react-app typescript-tea --template typescript

cd typescript-tea
npm start

```

1、添加 antd

```bash
yarn add antd @ant-design/icons
```

2、添加配置定制，样式定制

```bash
yarn add react-app-rewired customize-cra babel-plugin-import less less-loader
```

`react-app-rewired` 替换 `react-script`

```js
 "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
  }
```

创建 `config-overrides.js`, 获得 antd 组件的按需引用和主题定制的功能

```js
const { override, fixBabelImports, addLessLoader } = require("customize-cra");
const darkThemeVars = require("antd/dist/dark-theme");

module.exports = override(
  fixBabelImports("import", {
    libraryName: "antd",
    libraryDirectory: "es",
    style: true
  }),
  addLessLoader({
    lessOptions: {
      javascriptEnabled: true,
      modifyVars: {
        hack: `true;@import "${require.resolve(
          "antd/lib/style/color/colorPalette.less"
        )}";`,
        ...darkThemeVars,
        "@primary-color": "#02b875"
      }
    }
  })
);

```
