# 本地化定制icon组件

> 背景：iView 的图标使用开源项目 [ionicons](http://ionicons.com/), 但是项目开发中，有时需要一些定制icon, 这时如果能本地部署一份属于项目的icon, 这也能减少对图片的处理，减少打包体积，方便开发

![](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/project-note/icon/myicon.png)

本地定制icon组件，采用的是阿里系的[iconfont](http://iconfont.cn)

## 写在前面

> 本文是在使用[react ant-design](https://ant.design/components/icon-cn/)，进行使用总结。

**参考**

- [antd-icon本地部署](https://github.com/ant-design/antd-init/tree/master/examples/local-iconfont)

## 开发简介

### [iconfont](http://iconfont.cn)

>新建项目时FontClass/Symbol前缀和Font Family字段的填写需要注意，前缀不要和项目所使用的UI库字体前缀一样，预防命名冲突

1、利用github账户或者注册账户，创建账户
2、寻找或创建icon, 添加入项目（提前创建项目）
3、项目编辑
4、下载图标至本地（文件名里面带有`demo`, 只是demo展示，放入项目中删除）

### 组件开发

> 使用所属库语法，开发本地icon组件

- 将`.css` 文件样式放入组件，注意icon资源路径的修改
- 组件命名语法冲突

### 参考使用

- [FireLeaf-React-Scaffold](https://github.com/NARUTOne/FireLeaf-React-Scaffold)
- [FireLeaf-Vue-Scaffold](https://github.com/NARUTOne/FireLeaf-Vue-Scaffold)
