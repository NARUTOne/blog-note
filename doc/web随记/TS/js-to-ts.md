# JS项目TS改造

> 纯JS项目（node项目、工具项目等）需要使用TS，加入类型检查。包含一些：目录结构调整、ts-eslint配置、tsconfig配置、测试jest、打包编译等

参考:

- [TS官网-JavaScript迁移](https://www.tslang.cn/docs/handbook/migrating-from-javascript.html)
- [TypeScript、Rollup 搭建工具库](https://juejin.im/post/6844904035309322254)

## 基本调整

> 使用`三板斧`: 下载、创建、修改

### install

> 下载`typescript`依赖

```bash
npm install typescript
```

### create

> 创建`tsconfig.json`, 指定检查范围

```js
  {
    "compilerOptions": {
        "outDir": "./dist",
        "allowJs": true,
        "target": "es5"
    },
    "include": [
        "./src/**/*"
    ]
  }
```

### update

> 将想要使用迁移到TS的目录、文件列出

- 修改`.js`后缀，改为`.ts`
- 每个ts文件的错误纠正，类型声明补充
- 为第三方 JavaScript 代码定义环境声明
  - 存在typescript类型定义，进行引入
  - 存在`@types/packagename`, 下载
  - 自定义声明 `packagename.d.ts`
- 非Javascript资源

  ```js
    declare module '*.css';
    declare module '*.png';
  ```

## eslint

```javascript
npm i --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

在根目录新建 `.eslintrc.js` 配置文件，并设置以下配置：

```javascript
module.exports = {
  root: true,
  parser: "@typescript-eslint/parser",
  plugins: ["@typescript-eslint"],
  extends: ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
};
```

script命令

```javascript
"scripts": {
  "lint": "eslint src",
}
```

### Prettier 配置

首先安装 Prettier 所需要的依赖：

```javascript
npm i  prettier eslint-config-prettier --save-dev
```

```javascript
{
  "extends": [
    "plugin:@typescript-eslint/recommended",
    // 用于关闭 ESLint 相关的格式规则集，具体可查看 https://github.com/prettier/eslint-config-prettier/blob/master/index.js
    "prettier",
    // 用于关闭 @typescript-eslint/eslint-plugin 插件相关的格式规则集，具体可查看 https://github.com/prettier/eslint-config-prettier/blob/master/%40typescript-eslint.js
    "prettier/@typescript-eslint",
  ]
}
```

配置完成后，可以通过[命令行接口](https://prettier.io/docs/en/cli.html)运行 Prettier:

```javascript
"scripts": {
  "prettier": "prettier src test --write",
},
```

## 编译

### tsc

> typescript 内置的编译工具

```json
{
  "scripts":{
    "build":"tsc",
    "watch":"tsc --watch"
  }
}
```

### gulp

```js
const gulp = require("gulp");
const ts = require("gulp-typescript");
const tsProject = ts.createProject("tsconfig.json");
const merge = require("merge2");

// 默认输出 CommonJS 规范到 dist 目录下
gulp.task("default", function () {
  const tsResult = tsProject.src().pipe(tsProject());
  
  return merge([
    tsResult.dts.pipe(gulp.dest("types")),
    tsResult.js.pipe(gulp.dest("dist")),
  ]);
});
```

## 报错

1、声明文件`.d.ts`

- 命名为 `global.d.ts` 或者 `vendor.d.ts` 文件, 非 `index.d.ts`

```js
declare var window: Window;
```

2、修改eslint

- 支持`.ts`扩展名 [Missing file extension “ts” import/extensions](https://stackoverflow.com/questions/59265981/typescript-eslint-missing-file-extension-ts-import-extensions)
