# css 简史
> 简述 `CSS` 的前世今生未来，如何成为和`html`,`javascript`并称为web前端的三大技术基石；又是如何出现我们相爱相杀（:boom:）的样式兼容问题的。

## 前世 :running:
> 自从在1991年，蒂姆·伯纳斯·李首次提出 HTML 的时候，也就是前端界面宇宙混沌初开，一片荒芜，并没有页面样式添加的方式。因此，群雄并起，逐鹿web端，出现了一大堆各式各样的样式方案，其中很多方案的特性为现在css的形成诞生提供了帮助。

### 浏览器标签样式

页面的样式设计，取决于使用的标签在浏览器的默认样式

```html
<MULTICOL COLS="3" GUTTER="25">
  <P><FONT SIZE="4" COLOR="RED">This would be some font broken up into columns</FONT></P>
</MULTICOL>
```

### 各种样式方案

- **RRP**：1993年6月，Robert Raisch在www-talk的邮件列表中给出了一个提案, 不支持层叠样式，单位决定于他们使用的上下文。
```
@BODY fo(fa=he,si=18)
```

- **PWP**：ViolaWWW 创建者Pei-Yuan Wei 创建了一个样式表语言，支持某种嵌套式的结构

```
<LINK REL="STYLE" HREF="URL_to_a_stylesheet">
...
(BODY fontSize=normal
      BGColor=white
      FGColor=black
  (H1   fontSize=largest
        BGColor=red
        FGColor=white)
)
```

- **FOSI**: 1987年，美国国防部CALS 团队创造了一门语言来定义 SGML 文档的样式，名为 FOSI

```
<outspec>
  <docdesc>
    <charlist>
      <font size="12pt" bckcol="white" fontcol="black">
    </charlist>
  </docdesc>
  <e-i-c gi="h1">\<font size="24pt" bckcol="red", fontcol="white"\></e-i-c>
  <e-i-c gi="h2">\<font size="20pt" bckcol="red", fgcol="white"\></e-i-c>
  <e-i-c gi="a"><font fgcol="red"></e-i-c>
  <e-i-c gi="cmd kbd screen listing example"><font style="monoser"></e-i-c></outspec>
```

- **DSSSL**: 基于函数式语言 Scheme 创建一门新的语言，基于强大的能力，能对文档进行任何你可以想到的转换

```
(define (create-heading heading-font-size)
  (make paragraph
    font-size: heading-font-size
    font-weight: 'bold))

(element h1 (create-heading 24pt))
(element h2 (create-heading 18pt))
```

- **PSL96**: 1996年版的“Presentation Specification Language”，从核心上看，PSL 与 CSS 很像

```
H1 {  fontSize: 20;}
LI {
  if (ChildNum(Self) == round(NumChildren(Parent) / 2 + 1)) then
    VertPos: Top = Parent.Top;  
    HorizPos: Left = LeftSib.Left + Self.Width;  
  else
    VertPos: Top = LeftSib.Actual Bottom;  
    HorizPos: Left = LeftSib.Left;  endif
}
```
支持如此的功能或许真的可以实现完美拆分内容和样式的代码。遗憾的是这门语言让人望而生畏，毕竟太易于扩展了。这意味着对于不同的浏览器很可能实现会很不一样。而且，它以学术界中的数篇论文发表出现，并没有 www-talk 邮件列表上进行研讨，要知道大部分的工作都是在这邮件列表里确定的。因此这门语言并没有被集成到主流的浏览器中。

- **CHSS**：1994年由 Håkon W Lie 提出，提出了样式权重渲染, 样式是可以重叠的

```
h1.font.size = 24pt 100%
h2.font.size = 20pt 40%
```
例如，如果之前的样式表定义h2的字体大小为30pt，有60%的权重，而加上这个样式表h2 20px的40%，根据权重将这两个值合到一起大概就是26pt。

### 浏览器支持

- Mosaic： 标签样式
- ViolaWWW：PWP
- 网景Netscape ：标签样式 ——> JSSS
- IE: 标签 ——> css
...

## 今生 :car:

### css 规范
> CSS 最值的一提的就是它的简单。它易于解析、编写和阅读。这种例子在互联网的历史里屡见不鲜。最终取胜的技术是那些初学者容易入门的，而不是那些给专家用的。

- css1.0: 1996年12月， Håkon Lie 简化他的提案，并与 Bert Bos 合作，W3C推出了CSS规范的第一个版本
- css2.0: 1998年
- css3: 2001年5月W3C开始进行CSS3标准的制定，到目前为止该标准还没有最终定稿。

```
h1{
  font-size: 20px;
  color: red;
}
a:hover {
  color: #e4393c;
}
```

### css预处理

- less: http://lesscss.cn/
- sass: https://www.sass.hk/
...


> css 目前一些浏览器发展的历史原因，有些兼容问题，可通过不同hack或一些技巧、插件[autoprefixer](https://github.com/postcss/autoprefixer)进行解决

## 未来 :rocket:

在未来的CSS中将会支持更多的属性以及函数，其中不乏有变量，嵌套，值计算等, 浏览器们也将统一规范，不再需要为了兼容而hack了
css 布局已经出现 flexbox, grid等友好的方式，也已经被css标准采纳，期待着css3的正式发布吧:smile:

[2017年前端有什么样变化？即将来临的2018有什么样的期待？- 知乎](https://www.zhihu.com/question/264551320)

## 参考

- [扒一扒 CSS 语言的诞生史](https://mp.weixin.qq.com/s?__biz=MzAwNTAzMjcxNg%3D%3D&mid=2651424749&idx=1&sn=f58bca144f50bff00ba7d1675cc8b8e7&scene=4)
- [CSS发展史](http://blog.csdn.net/lipeishenshen/article/details/50696271)
- [css简史](https://segmentfault.com/a/1190000011872815)
