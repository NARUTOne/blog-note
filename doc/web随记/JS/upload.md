# 前端学习随记の下载上传篇 —— 上传篇

>**闲言：** 上篇我们学习了一些下载的js前端方式，这篇来讲讲上传篇。这也是在项目需求中很常见的O(∩_∩)O。

>demo代码：https://github.com/NARUTOne/DownAndUp.git
>参考网址：
>[http://www.ruanyifeng.com/blog/2012/08/file_upload.html](http://www.ruanyifeng.com/blog/2012/08/file_upload.html)
>https://segmentfault.com/a/1190000006600936
>https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader
>https://developer.mozilla.org/zh-CN/docs/Web/API/File
>https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects

## 上传の方式 ##

### Form && Input ###

采用传统的form表单或异步ajax上传。至于以前通过iframe来进行异步上传，这里不详细介绍，想了解可以看
[http://www.ruanyifeng.com/blog/2012/08/file_upload.html](http://www.ruanyifeng.com/blog/2012/08/file_upload.html)。
``` markup
<form id="upload-form" action="./upload.do" 
onsubmit="return handleSubmit(this)" method="post" enctype="multipart/form-data" >
	<div class="filePicker">
        <label>点击选择文件</label>
        <input id="fileInput1" type="file" name="files" multiple="multiple">
    </div>
    <input type="submit" value="上传" />
 </form>
```
>**知识点：**
>1、input file标签设置accept属性进行文件选择过滤，该属性的值必须为一个逗号分割的列表,包含了多个唯一的内容类型声明：
> - 以 STOP 字符 (U+002E) 开始的文件扩展名。（例如：".jpg,.png,.doc"）
一个有效的 MIME 类型，但没有扩展名
- audio/* 表示音频文件 HTML5
- video/* 表示视频文件 HTML5
- image/* 表示图片文件

>2、设置multiple属性可以进行设置为多选。

>3、设置capture属性可以进行设置打开摄像拍照或者录像。multiple属性和capture属性不能同时生效。

```markup
Capture Image: 
<input type="file" accept="image/*" capture="camera"> 
Capture Audio: 
<input type="file" accept="audio/*" capture="microphone"> 
Capture Video: 
<input type="file" accept="video/*" capture="camcorder"> 
```

接着采用js来获取file对象，进行异步上传。

``` js
var fileInput1 = document.getElementById("fileInput1");
fileInput1.addEventListener('change', function(event) {
    var file = fileInput1.files[0];
    // 或file = fileInput1.files.item(0);
    console.log(file);
    document.getElementById('showFile1').innerHTML= file.name
}, false);

function handleSubmit(_this) {
	var form = $(_this);
	// mulitipart form,如文件上传类
    	var formData = new FormData();
			formData.append('files', $('#fileInput1')[0].files[0]);
//    var formData = new FormData(form[0]);
    $.ajax({
    type: form.attr('method'),
    url: form.attr('action'),
    data: formData,
    mimeType: "multipart/form-data",
    contentType: false,
    cache: false,
    processData: false
    }).success(function (res) {
    //成功提交
    console.log(res)
    document.getElementById('result').innerHTML= '上传成功'
    }).error(function (jqXHR, textStatus, errorThrown) {
    //错误信息
    document.getElementById('result').innerHTML= '上传失败'
    });
    
    return false;
}
```
>**注意：**预防form表单提交的跳转action。

>file对象：
>- lastModifiedDate：文件对象最后修改的日期
- name：文件名,只读字符串,不包含任何路径信息.
- size：文件大小,单位为字节,只读的64位整数.
- type：MIME类型,只读字符串,如果类型未知,则返回空字符串.

>type属性判断用户的文件类型不太准确，用户会改变后缀名；size可以做大小限制判断。

### HTML5 拖拽操作文件 ###

![](http://upload-images.jianshu.io/upload_images/2455352-2981df91b2eb4a07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>HTML5拖拽知识可以参考我以前的拖拽文章：[http://www.jianshu.com/p/6cd34dfe87fd](http://www.jianshu.com/p/6cd34dfe87fd)。
>对文件的操作还采用了一些FileReader API的知识，这个等会下面会简单介绍哈:-D。

```html
<div class='demo-box'>    		
	<div class="demo-content">
		<a href="javascript:;">选择图片文件拖拽至下方框中，实现拖拽上传及图片预览</a>
        <div id="dropbox" class="dropbox">
            <div class="area"></div>
        </div>
        <div id="preview"></div>
    </div>
</div>
```
```js
var dropbox = document.getElementById("dropbox");
var preview = document.getElementById("preview");

dropbox.addEventListener("dragenter", function(e){
    e.stopPropagation();
      e.preventDefault();
}, false);

dropbox.addEventListener("dragover", function(e){
    e.stopPropagation();
      e.preventDefault();
}, false);

dropbox.addEventListener("drop", function(e){
    e.stopPropagation();
      e.preventDefault();

      var dt = e.dataTransfer;
      var files = dt.files;//获取文件
      
      for (var i = 0; i < files.length; i++) {
        var file = files[i];
        var imageType = /^image\//;

        if ( !imageType.test(file.type) ) {
              continue;
        }
        
        // 填充选择的图片到展示区
        var img = document.createElement("img");
        img.classList.add("obj");
        img.file = file;
        preview.appendChild(img);
        
        // 读取File对象中的内容
        var reader = new FileReader();
        reader.onload = (function(aImg) { 
          return function(e) { 
            aImg.src = e.target.result; 
          }; 
        })(img);
        reader.readAsDataURL(file);
    }
}, false);
```

## FileReader API　##

>使用FileReader对象,web应用程序可以异步的读取存储在用户计算机上的文件(或者原始数据缓冲)内容,可以使用File对象或者Blob对象来指定所要处理的文件或数据.其中File对象可以是来自用户在一个< input >元素上选择文件后返回的FileList对象,也可以来自拖放操作生成的 DataTransfer对象,还可以是来自在一个HTMLCanvasElement上执行mozGetAsFile()方法后的返回结果。

### DataURI && URL ###

>**URL** 是uniform resource locator的缩写，在web中的每一个可访问资源都有一个URL地址，例如图片，HTML文件，js文件以及style sheet文件，我们可以通过这个地址去download这个资源。其实URL是URI的子集，URI是uniform resource identifier的缩写。
>**URI** 是用于获取资源，包括其附加的信息的一种协议。附加信息可能是地址，也可能不是地址，如果是地址，那么这时URI就变成URL了。
>**注意的是data URI不是URL，因为它并不包含资源的公共地址。**

**DataURI**
DataURI对象，允许将一个小文件进行编码后嵌入到另外一个文档里。
```
data:[<MIME type>][;charset=<charset>][;base64],<encoded data>
eg: data:image/png;base64,xxxxxxxxxxxxx
```
整体可以视为三部分，即声明：参数+数据，逗号左边的是各种参数，右边的是数据。

通过FileReader 的readAsDataURL方法获得：
```js
//读取File对象中的内容
var reader = new FileReader();
reader.onload = (function(aImg) { 
  return function(e) { 
    aImg.src = e.target.result; //result属性中将包含一个data: URL格式的字符串以表示所读取文件的内容.
  }; 
})(img);
reader.readAsDataURL(file);
```

有时候我们需要将DataURI对象转blob对象：
```js
/**
 * dataURI 转 blob
 * @param {Object} dataURI
 */
function dataURItoBlob(dataURI) {
    var arr = dataURI.split(','), mime = arr[0].match(/:(.*?);/)[1],
    bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
    while(n--){
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new Blob([u8arr], {type:mime});
}
```
**URL对象**
我们除了可以使用base64字符串作为内容的DataURI将一个文件嵌入到另外一个文档里，还可以使用URL对象。URL对象用于生成指向File对象或Blob对象的URL。

```js
var objecturl = window.URL.createObjectURL(file);
```
静态方法会创建一个 DOMString，它的 URL 表示参数中的对象。这个 URL 的生命周期和创建它的窗口中的 document 绑定。这个新的URL 对象表示着指定的 File 对象或者 Blob 对象。

```js
window.URL.revokeObjectURL(objecturl)
```
静态方法用来释放一个之前通过调用 window.URL.createObjectURL() 创建的已经存在的 URL 对象。当你结束使用某个 URL 对象时，应该通过调用这个方法来让浏览器知道不再需要保持这个文件的引用了

对象URL来显示图片：
```js
window.URL = window.URL || window.webkitURL;

var img = document.createElement("img");
img.src = window.URL.createObjectURL(blob);//二进制对象
img.height = 60;
img.onload = function(e) {
    window.URL.revokeObjectURL(this.src);
}
document.body.appendChild(img);
```

>FileReader API详解:https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader

上传实例：以二进制流上传文件
```js
var fileInput = document.getElementById("fileInput");
fileInput.addEventListener('change', function(event) {
    var file = fileInput.files[0];
    if (file) {
        var reader = new FileReader();  
        var xhr = new XMLHttpRequest();
        xhr.onprogress=function(e){
            var percentage = Math.round((e.loaded * 100) / e.total);
            console.log("percentage:"+percentage);
        }
        xhr.onload=function(e){
            console.log("percentage:100");
        }
        xhr.open("POST", "这里填写服务器地址");  
        reader.onload = function(evt) {
            xhr.send(evt.target.result);
        };
        reader.readAsBinaryString(file);
    }
});         
```

**blob对象**
>BLOB (binary large object)，二进制大对象，是一个可以存储二进制文件的容器。

>创建Blob对象的方法有几种，可以调用Blob构造函数，还可以使用一个已有Blob对象上的slice()方法切出另一个Blob对象，还可以调用canvas对象上的toBlob方法。

Blob对象有两个只读属性：

- size：二进制数据的大小，单位为字节。
- type：二进制数据的MIME类型，全部为小写，如果类型未知，则该值为空字符串。在Ajax操作中，如果xhr.responseType设为blob，接收的就是二进制数据。


** blob创建 **

1）构成函数
Blob构造函数，接受两个参数。第一个参数是一个包含实际数据的数组，第二个参数是数据的类型，这两个参数都不是必需的。数组元素可以是任意多个的ArrayBuffer，ArrayBufferView (typed array)， Blob，或者 DOMString对象。例如：

```js
var arr = ['<h1>hello world</h1>'];
var blob = new Blob(arr, { "type" : "text/xml" }); // the blob
console.log(blob);
```

如果在浏览器端js生成的内容，想让浏览器进行下载要如何办到呢？DataURI可以实现这个效果，但是DataURI的文件类型被限制了，我们这里可以变通一下实现blob对象。

```js
<a id="aLink">下载</a>
<script type="text/javascript">
    function downloadFile (el, fileName, content) {
        var aLink = document.querySelector(el);
        var blob = new Blob([content]);                
        aLink.download = fileName;
        aLink.href = URL.createObjectURL(blob);
    }
    document.querySelector('#aLink').addEventListener('click',function () {
        downloadFile('#aLink', 'hello.txt', '<h1>hello world</h1>');
    })
</script>
```
2)Blob对象的slice方法生成blob对象
Blob对象的slice方法，将二进制数据按照字节分块，返回一个新的Blob对象。

```js
var newBlob = oldBlob.slice(startingByte, endindByte);
```
大文件分割上传:
```js
function upload(blobOrFile) {
  var xhr = new XMLHttpRequest();
  xhr.open('POST', '/server', true);
  xhr.onload = function(e) { ... };
  xhr.send(blobOrFile);
}

document.querySelector('input[type="file"]').addEventListener('change', function(e) {
  var blob = this.files[0];

  const BYTES_PER_CHUNK = 1024 * 1024; // 1MB chunk sizes.
  const SIZE = blob.size;

  var start = 0;
  var end = BYTES_PER_CHUNK;

  while(start < SIZE) {
    upload(blob.slice(start, end));

    start = end;
    end = start + BYTES_PER_CHUNK;
  }
}, false);
```
##终语##
>文件上传下载，暂时就讲这么多哈O(∩_∩)O，智商知识技能有限，也就只能模仿学习到这哈。欢迎大家留言交流，互相进步(✿◡‿◡)。
>这里给个上传应用的例子（大神制作）：[前端实现文件的断点续传](http://web.jobbole.com/88384/)。