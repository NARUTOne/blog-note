# 前端工程化

> 在日常开发中制订一个规范化的前端工作流，很好地规范统一项目的模块化开发和前端资源，让代码的维护和互相协作更加容易更加方便，令前端开发自动化成为一种习惯。同时，让大家能够释放生产力，提高开发效率，更好更快地完成团队开发以及项目后期维护和扩展。

## 背景

随着前端开发复杂度的日益增加，各种各样适应业务的组件框架也层出不穷。同时，我们面临业务规模的快速发展和工程师团队的不断扩张，Web 业务日益复杂化和多元化，前端开发已经由以 WebPage 模式为主转变为以 WebApp 模式为主了。现在的前端项目，再也不是过去那种切个图 + 搞几个 jQuery 插件就能实现业务所有功能了

- 如何进行高效的多人协作？
- 如何保证项目的可维护性？
- 如何提高项目的开发质量？

## 前端发展

### 1、刀耕火种时代

适合小项目，不分前后端，页面由JSP、PHP等在服务端生成，浏览器负责展现
![web 1.0](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/web-1.0.png)

### 2、后端为主的MVC时代

为了降低复杂度，以后端为出发点，有了Web Server层的架构升级，比如Structs、Spring MVC等。
![web 1.0](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/web-mvc.png)

**前两个阶段存在的问题**：

- 前端开发重度依赖开发环境。
- 前后端职责依旧纠缠不清，可维护性越来越差。

### 3、Ajax 带来的 SPA 时代

2005年Ajax正式提出，前端开发进入SPA（Single Page Application 单页面应用）时代
![web 1.0](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/web-spa.png)

**Ajax时代面临的挑战**：

- 前后端接口的约定
- 前端开发的复杂度控制。SPA应用大多以功能交互型为主，大量 JS 代码的组织，与 View 层的绑定等，都不是容易的事情。

### 4、前端为主的 MVC、MV* 时代

> [MVC，MVP 和 MVVM 的图示](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)

为了降低前端开发复杂度，Backbone、EmberJS、KnockoutJS、AngularJS、React、Vue等大量前端框架涌现
![web 1.0](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/web-mvc.png)

### 5、Node 带来的全栈时代

随着Node.js的兴起，为前端开发带来一种新的开发模式: 中途岛模式
![web 1.0](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/web-node.png)

**后两个步骤带来的好处**:

- 前后端职责很清晰。
- 前端开发的复杂度可控，通过合理的分层，让项目更可维护。
- 部署相对独立，产品体验可以快速改进。

## 工程化演进

- 工具化：个人，利用一些工具、插件、组件库来进行业务开发，提升效率
- 规范化：初级团队（10），多人协同，约定规范：技术架构 + git + 代码格式 + 前端项目构建等规范
- 流程化：中级团队（20），规范落地后，需要流程标准化去构建、检测、发布、监控等
- 中台化：高级团队（100往上），抽象、抽取研发过程中的技能，分发到具体团队，进行定制化部分抽象。

## 工程化尝试

### 规范

1、编码规范

- 编码规范：eslint + stylelint
- 变量、样式命名规范

2、开发流程

- 项目构建，目录结构，文件命名约定
- 项目打包，维护管理和控制
- code review 机制规范

3、文档完善

- 前端、后台接口文档规范
- 需求文档规范
- 前端（部分）逻辑交互文档规范

### 工具

- git版本控制、分支管理
  - commit 描述
  - merge request 需要进行code review
  - 提交代码前，需要进行pull，对比最新代码

- vscode (webstorm、sublime等其他IDE)
  - 安装 eslint + stylelint 进行代码规范检查
  - 设置一致的代码缩进方式 （2个空格）
  - 对于不同技术栈，安装相关技术语法校验插件 （Vue, React, TS）

### 技术框架

> 推荐

1、样式

less 、sass 、css modules等

2、库（框架）

- Vue : Vue + element(iview) + vuex + vue-router 等其他社区优秀库
- React : React + antd + redux(mobx) + react-router 等其他社区优秀库
- Angular : Angular + angular-cli (本次不做重点)

3、构建工具

- Vue : vue-cli
- React : create react app 、 antd-pro

4、服务器渲染

> 首屏加载速度快、SEO优化; 所见即所得（html）

Next.js(React) 、 Nuxt.js(Vue)

5、 测试

引入 组件、方法的单元测试，提高代码质量

jest，mocha等

6、 自动化打包

- webpack
- gulp
- grunt (不推荐)

7、 组件库

> 服务于产品设计，基于模块化的解决方案

- React
  - antd
  - React-Bootstrap
- Vue
  - element
  - iview

8、可视化

- echarts
- d3JS
- antv
- TreeJS

9、其他

地图、动效（动画）、富文本等

### 自建

> 根据自身业务需求，设计UI，开发符合自己的一套工程体系。
