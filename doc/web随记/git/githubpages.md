# github-pages

> github pages 建站 🚀

[https://pages.github.com/](https://pages.github.com/)

## 萌生念头

为了更好地学习和记录自己的前端经历，搭建一个便捷、稳定的个人站点挺有必要的；而通过GitHub Pages来展示自己的作品，更是不错的选择。

>**注意：** 需要保证已经安装好git并且已经关联到github远程平台账户

## 搭建静态网页

> 分两种方式介绍

### one

创建 repository, 命名：name.github.io

### two

创建 branch, 命名：gh-pages；setting 中 GitHub  Pages配置 分支

```sh
git checkout --orphan gh-pages
```

> **注意**将项目的展示页面文件放在项目仓库的gh-pages分支的根目录下，并且命名为index.html，必须以html为扩展名。
> 同时如果使用webpack这类的打包工具，注意文件的引用url, 需要配置上完全路径。

## 搭建个人网站

> 基于 jekyll 和  hexo

### 参考

- [https://github.com/uolcano/blog/issues/11](https://github.com/uolcano/blog/issues/11)
