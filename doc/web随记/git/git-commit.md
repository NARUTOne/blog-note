# commit message

> 代码提交信息规范

参考：

- [优雅的提交你的 Git Commit Message](https://juejin.im/post/5afc5242f265da0b7f44bee4)
- [前端工程化中的代码规范和commit规范](https://zhuanlan.zhihu.com/p/71143472)

## 格式

> Angular 团队规范，衍生 [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/)

```sh
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

git commit 命令带出的 vim 界面填写的最终结果应该类似如上这个结构, 大致分为三个部分(使用空行分割):

- 标题行: 必填, 描述主要修改类型和内容
- 主题内容: 描述为什么修改, 做了什么样的修改, 以及开发的思路等等
- 页脚注释: 放 Breaking Changes 或 Closed Issues

分别由如下部分构成:

- type: commit 的类型
- feat: 新特性
- fix: 修改问题
- refactor: 代码重构
- docs: 文档修改
- style: 代码格式修改, 注意不是 css 修改
- test: 测试用例修改
- chore: 其他修改, 比如构建流程, 依赖管理.
- scope: commit 影响的范围, 比如: route, component, utils, build...
- subject: commit 的概述, 建议符合  50/72 formatting
- body: commit 具体修改内容, 可以分为多行, 建议符合 50/72 formatting
- footer: 一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接.

符合规范的 commit message, 就好像是一份邮件.

## 模板

如果你只是个人的项目, 或者想尝试一下这样的规范格式, 那么你可以为 git 设置 commit template, 每次 git commit 的时候在 vim 中带出, 时刻提醒自己:
修改 ~/.gitconfig, 添加:

```sh
[commit]
template = ~/.gitmessage
```

复制代码新建 ~/.gitmessage 内容可以如下:

```sh
# head: <type>(<scope>): <subject>
# - type: feat, fix, docs, style, refactor, test, chore
# - scope: can be empty (eg. if the change is a global or difficult to assign to a single component)
# - subject: start with verb (such as 'change'), 50-character line
#
# body: 72-character wrapped. This should answer:
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# footer:
# - Include a link to the ticket, if any.
# - BREAKING CHANGE
#
```

## 工具

> 通过工具生成和约束commit message 规范

### commitizen/cz-cli

> [http://commitizen.github.io/cz-cli/](http://commitizen.github.io/cz-cli/), 提供的 git cz 命令替代我们的 git commit 命令, 帮助我们生成符合规范的 commit message.

#### 全局安装

```sh
npm install -g commitizen cz-conventional-changelog
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

复制代码主要, 全局模式下, 需要 ~/.czrc 配置文件, 为 commitizen 指定 Adapter.

#### 项目级安装

`npm install -D commitizen cz-conventional-changelog`

复制代码package.json中配置:

```json
"script": {
    ...,
    "commit": "git-cz",
},
 "config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog"
    }
  }
```

执行

`git cz or npm run commit`

#### 自定义 Adapter

也许 Angular 的那套规范我们不习惯, 那么可以通过指定 Adapter cz-customizable 指定一套符合自己团队的规范.

```sh
npm i -g cz-customizable
# or
npm i -D cz-customizable
```

修改 .czrc 或 package.json 中的 config 为:

```json
{ "path": "cz-customizable" }
// or
"config": {
  "commitizen": {
    "path": "node_modules/cz-customizable"
  }
}
```

同时在~/ 或项目目录下创建 .cz-config.js 文件, 维护你想要的格式: 比如我的配置文件: demo/.cz-config

#### Commitlint: 校验message

[commitlint](https://github.com/conventional-changelog/commitlint) 可以帮助我们 lint commit messages, 如果我们提交的不符合指向的规范, 直接拒绝提交, 比较狠.
推荐 `@commitlint/config-conventional` (符合 Angular团队规范).

```sh
npm i -D @commitlint/config-conventional @commitlint/cli

# 自定义 Adapter

npm i -D commitlint-config-cz @commitlint/cli
```

同时需要在项目目录下创建配置文件 .commitlintrc.js, 写入:

```js
module.exports = {
  extends: [
    ''@commitlint/config-conventional''
  ],
  rules: {
  }
};

// 自定义 Adapter

module.exports = {
  extends: [
    'cz'
  ],
  rules: {
  }
};
```

## 结合 Husky

> 校验 commit message 的最佳方式是结合 git hook, 所以需要配合 [Husky](https://github.com/typicode/husky).

`npm i husky@next`

package.json 中添加:

```json
"husky": {
  "hooks": {
    ...,
    "commit-msg": "commitlint -e $GIT_PARAMS"
  }
},
```

## standard-version: 自动生成 CHANGELOG

安装使用:

`npm i -S standard-version`

复制代码

package.json 配置:

```json
"scirpt": {
    ...,
    "release": "standard-version"
}
```
