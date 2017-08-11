# web下的高度、位置

## 屏幕、浏览器、页面的高度宽度

> NARUTOne

> 相信各位web开发狮们，在项目中为了搭建漂亮酷炫的页面，经常会遇到需要获取各种高宽吧，屏幕的、浏览器的、页面的。应该也有些人对这些各种高宽有过疑惑吧。这篇文章我将简单介绍下这些宽高，发发身份证,最后加上点元素大小及位置介绍O(∩_∩)O~。

>老规矩（参考链接）：[http://www.cnblogs.com/polk6/p/5051935.html](http://www.cnblogs.com/polk6/p/5051935.html)

>[http://www.cnblogs.com/autismtune/p/5264589.html](http://www.cnblogs.com/autismtune/p/5264589.html)

### 屏幕

设备的可视大小。

![](http://upload-images.jianshu.io/upload_images/2455352-3e44605852e45b12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- screen.height ：屏幕高度。

- screen.width ：屏幕宽度。

- screen.availHeight ：屏幕可用高度。即屏幕高度减去上下任务栏后的高度，可表示为软件最大化时的高度。

- screen.availWidth ：屏幕可用宽度。即屏幕宽度减去左右任务栏后的宽度，可表示为软件最大化时的宽度。

- 任务栏高/宽度 ：可通过屏幕高/宽度 减去 屏幕可用高/宽度得出。如：任务栏高度 = screen.height - screen.availHeight 。
### 浏览器

浏览器窗口的大小，则是指在浏览器窗口中看到的那部分网页面积，又叫做viewport（视口）。

![](http://upload-images.jianshu.io/upload_images/2455352-314764e5005eaafd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- window.outerHeight ：浏览器高度。

- window.outerWidth ：浏览器宽度。

- window.innerHeight ：浏览器内页面可用高度；此高度包含了水平滚动条的高度(若存在)。可表示为浏览器当前高度去除浏览器边框、工具条后的高度。

- window.innerWidth ：浏览器内页面可用宽度；此宽度包含了垂直滚动条的宽度(若存在)。可表示为浏览器当前宽度去除浏览器边框后的宽度。

- 工具栏高/宽度 ：包含了地址栏、书签栏、浏览器边框等范围。如：高度，可通过浏览器高度 - 页面可用高度得出，即：window.outerHeight - window.innerHeight。

### body页面

一张网页的全部面积，就是它的大小。通常情况下，网页的大小由内容和CSS样式表决定。

很显然，如果网页的内容能够在浏览器窗口中全部显示（也就是不出现滚动条），那么网页的大小和浏览器窗口的大小是相等的。如果不能全部显示，则滚动浏览器窗口，可以显示出网页的各个部分。

![](http://upload-images.jianshu.io/upload_images/2455352-0cd7158c5d2f5ea8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- body.offsetHeight ：body总高度。

- body.offsetWidth ：body总宽度。出现滚动条后，与scrollHeight一致。

- body.clientHeight ：body展示的高度；表示body在浏览器内显示的区域高度。

- body.clientWidth ：body展示的宽度；表示body在浏览器内显示的区域宽度。

- 滚动条高度/宽度 ：如高度，可通过浏览器内页面可用高度 - body展示高度得出，即window.innerHeight - body.clientHeight。

**合并到一张图**

![](http://upload-images.jianshu.io/upload_images/2455352-0b31a8f6b3594e05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 进阶篇 —— 元素大小、位置

### 盒子模型

![](http://upload-images.jianshu.io/upload_images/2455352-921aaf9208e96b4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

CSS3中新增了一种盒模型计算方式：box-sizing熟悉。盒模型默认的值是content-box, 新增的值是padding-box和border-box，几种盒模型计算元素宽高的区别如下：

- content-box（默认）

布局所占宽度Width：

Width = width + padding-left + padding-right + border-left + border-right

布局所占高度Height:

Height = height + padding-top + padding-bottom + border-top + border-bottom

- padding-box

布局所占宽度Width：

Width = width(包含padding-left + padding-right) + border-top + border-bottom

布局所占高度Height:

Height = height(包含padding-top + padding-bottom) + border-top + border-bottom

- border-box

布局所占宽度Width：

Width = width(包含padding-left + padding-right + border-left + border-right)

布局所占高度Height:

Height = height(包含padding-top + padding-bottom + border-top + border-bottom)

### 元素的决对位置

>网页元素的绝对位置，指该元素的左上角相对于整张网页左上角的坐标。这个绝对位置要通过计算才能得到。

>首先，每个元素都有offsetTop和offsetLeft属性，表示该元素的左上角与父容器（offsetParent对象）左上角的距离。所以，只需要将这两个值进行累加，就可以得到该元素的绝对坐标。

![](http://upload-images.jianshu.io/upload_images/2455352-6ccda43fa8fefe16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面两个函数可以用来获取绝对位置的横坐标和纵坐标。

	　　function getElementLeft(element){
	　　　　var actualLeft = element.offsetLeft;
	　　　　var current = element.offsetParent;
	
	　　　　while (current !== null){
	　　　　　　actualLeft += current.offsetLeft;
	　　　　　　current = current.offsetParent;
	　　　　}
	
	　　　　return actualLeft;
	　　}
	
	　　function getElementTop(element){
	　　　　var actualTop = element.offsetTop;
	　　　　var current = element.offsetParent;
	
	　　　　while (current !== null){
	　　　　　　actualTop += current.offsetTop;
	　　　　　　current = current.offsetParent;
	　　　　}
	
	　　　　return actualTop;
	　　}

>由于在表格和iframe中，offsetParent对象未必等于父容器，所以上面的函数对于表格和iframe中的元素不适用。


### 获取网页元素的相对位置

网页元素的相对位置，指该元素左上角相对于浏览器窗口左上角的坐标。

有了绝对位置以后，获得相对位置就很容易了，只要将绝对坐标减去页面的滚动条滚动的距离就可以了。滚动条滚动的垂直距离，是document对象的scrollTop属性；滚动条滚动的水平距离是document对象的scrollLeft属性。

![](http://upload-images.jianshu.io/upload_images/2455352-3308423fa28523f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对上一节中的两个函数进行相应的改写：

	　　function getElementViewLeft(element){
	　　　　var actualLeft = element.offsetLeft;
	　　　　var current = element.offsetParent;
	
	　　　　while (current !== null){
	　　　　　　actualLeft += current.offsetLeft;
	　　　　　　current = current.offsetParent;
	　　　　}
	
	　　　　if (document.compatMode == "BackCompat"){
	　　　　　　var elementScrollLeft=document.body.scrollLeft;
	　　　　} else {
	　　　　　　var elementScrollLeft=document.documentElement.scrollLeft; 
	　　　　}
	
	　　　　return actualLeft-elementScrollLeft;
	　　}
	
	　　function getElementViewTop(element){
	　　　　var actualTop = element.offsetTop;
	　　　　var current = element.offsetParent;
	
	　　　　while (current !== null){
	　　　　　　actualTop += current. offsetTop;
	　　　　　　current = current.offsetParent;
	　　　　}
	
	　　　　 if (document.compatMode == "BackCompat"){
	　　　　　　var elementScrollTop=document.body.scrollTop;
	　　　　} else {
	　　　　　　var elementScrollTop=document.documentElement.scrollTop; 
	　　　　}
	
	　　　　return actualTop-elementScrollTop;
	　　}

>scrollTop和scrollLeft属性是可以赋值的，并且会立即自动滚动网页到相应位置，因此可以利用它们改变网页元素的相对位置。另外，element.scrollIntoView()方法也有类似作用，可以使网页元素出现在浏览器窗口的左上角。

### 获取元素位置的快速方法

除了上面的函数以外，还有一种快速方法，可以立刻获得网页元素的位置。

那就是使用getBoundingClientRect()方法。它返回一个对象，其中包含了left、right、top、bottom四个属性，分别对应了该元素的左上角和右下角相对于浏览器窗口（viewport）左上角的距离。

所以，网页元素的相对位置就是

　　var X= this.getBoundingClientRect().left;

　　var Y =this.getBoundingClientRect().top;

再加上滚动距离，就可以得到绝对位置

　　var X= this.getBoundingClientRect().left+document.documentElement.scrollLeft;

　　var Y =this.getBoundingClientRect().top+document.documentElement.scrollTop;

目前，IE、Firefox 3.0+、Opera 9.5+都支持该方法，而Firefox 2.x、Safari、Chrome、Konqueror不支持。

## 基础知识

>在一些复杂的页面中经常会用JavaScript处理一些DOM元素的动态效果，这种时候我们经常会用到一些元素位置和尺寸的计算，浏览器兼容性问题也是不可忽略的一部分，要想写出预想效果的JavaScript代码，我们需要了解一些基本知识。

### clientHeight和clientWidth
- 用于描述元素内尺寸，是指 元素内容+内边距 大小，不包括边框（IE下实际包括）、外边距、滚动条部分
- 如果没有滚动条，即为元素设定的高度和宽度，如果出现滚动条，滚动条会遮盖元素的宽高，那么该属性就是其本来宽高减去滚动条的宽高

### offsetHeight和offsetWidth
- 用于描述元素外尺寸，是指 元素内容+内边距+边框，不包括外边距和滚动条部分及溢出部分
- 该属性和其内部的内容是否超出元素大小无关，只和本来设定的border以及width和height有关

### clientTop和clientLeft
- 返回内边距的边缘和边框的外边缘之间的水平和垂直距离，也就是左，上边框宽度

### offsetTop和offsetLeft
- 表示该元素的左上角（边框外边缘）与已定位的父容器（offsetParent对象）左上角的距离

### offsetParent
- 是指元素最近的定位（relative,absolute）祖先元素，递归上溯，找不到这样一个父级元素，那么当前元素的offsetParent就是body元素，如果没有祖先元素是定位的话，会返回null

### scrollWidth和scrollHeight
- 是元素的内容区域加上内边距加上溢出尺寸，当内容正好和内容区域匹配没有溢出时，这些属性与clientWidth和clientHeight相等
- ，需要注意的是，当元素其中内容没有超过其高度或者宽度的时候，该属性是取不到的。

### scrollLeft和scrollTop
- 是指元素滚动条位置，它们是**可写的**
- 元素被卷起的高度和宽度。

###　获取样式
- obj.style.*属性
- obj.currentstyle（IE）
- getComputedStyle(IE之外的浏览器)。

## event对象下的位置

### clientX和clientY

- 这对属性是当事件发生时，鼠标点击位置相对于浏览器（可视区）的坐标，即浏览器左上角坐标的（0,0），该属性以浏览器左上角坐标为原点，计算鼠标点击位置距离其左上角的位置，

- 不管浏览器窗口大小如何变化，都不会影响点击位置的坐标。

### screenX和screenY
- 是事件发生时鼠标相对于屏幕的坐标，以设备屏幕的左上角为原点，事件发生时鼠标点击的地方即为该点的screenX和screenY值

### offsetX 和 offsetY

- 这一对属性是指当事件发生时，鼠标点击位置相对于该事件源的位置，即点击该div，以该div左上角为原点来计算鼠标点击位置的坐标

- Firefox不支持该属性，Firefox中与此属性相对应的概念是，event.layerX和event.layerY,所以需要兼容浏览器时，获取鼠标点击位置相对于事件源的坐标的兼容写法为var disX=event.offsetX||event.layerX

### pageX和pageY

- 该属性是事件发生时鼠标点击位置相对于页面的位置。
- 通常浏览器窗口没有出现滚动条时，该属性和event.clientX及event.clientY是等价的，但是当浏览器出现滚动条的时候，pageX通常会大于clientX，因为页面还存在被卷起来的部分的宽度和高度，