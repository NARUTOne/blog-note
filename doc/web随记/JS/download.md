# 前端学习随记の下载上传篇 —— 下载篇

>**闲言：** 懒懒懒！！！好长时间不写文章了，一直出差魔都，学习一些其他知识开发项目，也时不时的空闲时间深度学习一些前端基础知识。好吧，不找理由，就是懒哇哈哈(✿◡‿◡)。

>demo代码：https://github.com/NARUTOne/DownAndUp.git

> 参考传送门：
> [http://www.cnblogs.com/voiphudong/p/3284724.html](http://www.cnblogs.com/voiphudong/p/3284724.html)
>[https://scarletsky.github.io/2016/07/03/download-file-using-javascript/#参考资料](https://scarletsky.github.io/2016/07/03/download-file-using-javascript/#参考资料)
> [https://developer.mozilla.org/zh-CN/docs/Web/API/FileSystem](https://developer.mozilla.org/zh-CN/docs/Web/API/FileSystem)

###引言###

上传下载一直是项目上非常常见的需求，也是对于我这种前端小白来说一直隐隐犯怵的地方，不太熟悉，加上项目上用过一次，同事朋友也问过遇到过，所以打算好好恶补这块↖(^ω^)↗。本篇先说下下载，上传留着下篇来说，欢迎关注留bug.

## 几种下载方式 ##

前后端下载方式原理简述：

1）**标准URL下载方式: **可以通过在web页面中嵌入 url超级链接，标准的HTTP GET请求。对于服务器端web根目录有一个test.zip的文件。此种方法的弊端是完全暴露了文件test.zip的网站路径，而且动态性不够灵活。

2)**通过服务器端脚本向浏览器方（stdout）输出二进制流的方式下载: **上述方法可以在服务器端 通过 脚本程序 download.php ，并且根据传入的查询字符串f=test.zip定位服务器上文件系统test.zip的路径，然后按二进制流的方式发送给客户端浏览器，那么客户端浏览器就会弹出一个“下载对话框”了。

### form 表单下载 ###

 POST的URL下载链接，可以通过设置form表单的action 以及 表单元素值来完成下载。

```js
function downLoad () {
  var form = document.createElement('form');   //定义一个form表单
  form.style = 'display:none';   //下面为在form表单中添加查询参数
  form.target = '';
  form.method = 'post';
  form.action = url;//下载接口

  var input1 = document.createElement('input');
  input1.type = 'hidden';
  Input1.name = 'exportData';
  input1.value = (new Date()).getMilliseconds(); 

  document.body.appendChild(form)  //将表单放置在web中
  form.appendChild(input1);   //将查询参数控件提交到表单上
  form.submit();   //表单提交
}
```

### iframe ###

iframe有一个src属性，其本质就是发送http请求，GET一个页面或者数据,可以通过一个动态生成的隐藏的iframe来得到下载的二进制文件。

```js
function download(){
  var IFrameRequest=document.createElement("iframe");
  IFrameRequest.id="IFrameRequest";
  IFrameRequest.src="/test.zip";
  IFrameRequest.style.display="none";
  document.body.appendChild(IFrameRequest);
}
```

### HTML5のdownload属性 ###

由于浏览器的安全策略的限制，我们通常只能通过一个额外的页面，访问某个文件的 url 来实现下载功能，但是这种用户体验非常不好。HTML 5 里面为 a 标签添加了一个 download 的属性，我们可以轻易的利用它来实现下载功能。

    <a href="http://somehost/somefile.zip" download="filename.zip">Download file</a>
> a标签添加 download 属性，我们点击这个链接的时候就会自动下载文件了~
顺便说下，download 的属性值是可选的，它用来指定下载文件的文件名。像上面的例子中，我们下载到本地的文件名就会是 filename.zip 拉，如果不指定的话，它就会是 somefile.zip 这个名字啦！

当然，我们还可以动态创建a标签实现下载：

```js
function download() {
  var a = document.createElement('a');
  var url = window.URL.createObjectURL(blob);
  var filename = 'what-you-want.txt';
  a.href = url;
  a.download = filename;
  a.click();
  window.URL.revokeObjectURL(url);
}
```

> window.URL 里面有两个方法：
createObjectURL 用 blob 对象来创建一个 object URL(它是一个 DOMString)，我们可以用这个 object URL 来表示某个 blob 对象，这个 object URL 可以用在 href 和 src 之类的属性上。
revokeObjectURL 释放由 createObjectURL 创建的 object URL，当该 object URL 不需要的时候，我们要主动调用这个方法来获取最佳性能和内存使用。
>当然，如果你没有blob对象的url,也可以不用window.URL，直接使用下载地址url。

### HTML5のFileSystem ###

> 由于这种方法目前大多数浏览器不支持，仅仅chrome支持，具体方式可以查看大神们的文章，我就不搬运了。(✿◡‿◡)
> [https://imququ.com/post/a-downloader-with-filesystem-api.html](https://imququ.com/post/a-downloader-with-filesystem-api.html)


### 其他方式 ###
>Window.open(url):这种方式在目前的网络环境及安全限制下被拦截已经不太好使了，主要用来是打开新窗口下载。

```js
var new_window = null;
function downLoad () {
  var isSure=confirm("是否下载")
  if (isSure){
    new_window = window.open()
    new_window.location.href = 'https://github.com/WZOnePiece/study-draggable/archive/master.zip';
    document.getElementById('result').innerHTML = "You downLoad OK!"
  }
  else{
    document.getElementById('result').innerHTML = "You pressed Cancel!"
  }
}
function closeDownLoad() {
  new_window  && new_window.close();
}

//使用a链接target='_blank'，则不会拦截下载
function AdownLoad() {
  var url = 'https://github.com/WZOnePiece/study-draggable/archive/master.zip';
  var a = document.createElement("a");
  a.setAttribute("href", url);
  a.setAttribute("target", "_blank");
  a.setAttribute("id", "camnpr");
  document.body.appendChild(a);
  a.click();
}
```

这里主要是用window.open 打开窗口，接着使用location.href = url进行重定向避免拦截进行下载。

## 补充知识 ##

### Blob 对象 ###

>Blob 全称是 Binary large object，它表示一个类文件对象，可以用它来表示一个文件。根据 MDN 上面的说法，File API 也是基于 blob 来实现的。

构建 blob 的方式就是通过服务器返回的文件来创建 blob ，采用File API及Fetch API。

```js
fetch('http://somehost/somefile.zip').then(res => res.blob().then(blob => {
  var a = document.createElement('a');
  var url = window.URL.createObjectURL(blob);
  var filename = 'myfile.zip';
  a.href = url;
  a.download = filename;
  a.click();
  window.URL.revokeObjectURL(url);
}))
```

>注意：Fetch API 目前浏览器支持不好，详情看[https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)
>File API的详细介绍：[https://developer.mozilla.org/zh-CN/docs/Using_files_from_web_applications](https://developer.mozilla.org/zh-CN/docs/Using_files_from_web_applications)
> 可以利用fetch配合reader对象来实现进度条功能

**限制**

1、不同浏览器对 blob 对象有不同的限制

Supported browsers
------------------

| Browser        | Constructs as | Filenames    | Max Blob Size | Dependencies |
| -------------- | ------------- | ------------ | ------------- | ------------ |
| Firefox 20+    | Blob          | Yes          | 800 MiB       | None         |
| Firefox < 20   | data: URI     | No           | n/a           | [Blob.js](https://github.com/eligrey/Blob.js) |
| Chrome         | Blob          | Yes          | [500 MiB][3]  | None         |
| Chrome for Android | Blob      | Yes          | [500 MiB][3]  | None         |
| Edge           | Blob          | Yes          | ?             | None         |
| IE 10+         | Blob          | Yes          | 600 MiB       | None         |
| Opera 15+      | Blob          | Yes          | 500 MiB       | None         |
| Opera < 15     | data: URI     | No           | n/a           | [Blob.js](https://github.com/eligrey/Blob.js) |
| Safari 6.1+*   | Blob          | No           | ?             | None         |
| Safari < 6     | data: URI     | No           | n/a           | [Blob.js](https://github.com/eligrey/Blob.js) |

2、构建完 blob 对象后才会转换成文件。

文件大点的话，要等到下载完文件之后才能构建 blob 对象，再转化成文件，页面显示上不太友好，没有提示。这时建议采用a标签，防止用户的错误认知造成重复点击下载，资源浪费。

## 服务器出错

- 1、window.open(url)

打开一个带错误信息的页面

- 2、window.location.href

页面将跳转到一个错误页面

- 3、iframe

用户感知不到任何变化

- 4、a标签

直接出现 下载失败

> 当然，如果response header里有content-disposition字段的话，浏览器都会下载一个带错误信息的文件。这时候，其实我们可以多发一个ajax/fetch请求，先检测下接口状态，然后再取做下载逻辑。但这样就对服务器造成了额外的开销。

**这样的体验都不太好，作为一个追求极致体验的程序猿，我们应该重新思考下，如何提升用户体验。**

##结语##
>O(∩_∩)O，这篇下载篇就这样简单介绍下，至于还有其他的方式，欢迎留言告知；代码及描述上有bug也希望不吝指出。谢谢哈！