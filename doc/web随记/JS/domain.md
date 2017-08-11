# 跨域

> NARUTOne

> 引言：项目开发，目前的开发方式的趋势主流是前后端分离进行开发，不同部分都交给专业人士，这样项目的开发速度及效率都将大大提高。但项目联调阶段，这也大大增加了跨域问题。

>参考：[https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
[https://segmentfault.com/a/1190000000718840](https://segmentfault.com/a/1190000000718840)
[http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)

## 浏览器同源政策

浏览器安全的基石是"同源政策"（same-origin policy）。1995年，同源政策由 Netscape 公司引入浏览器。

**三大相同**

- 协议相同
- 域名相同
- 端口相同

同源政策的目的，是为了保证用户信息的安全，防止恶意的网站窃取数据。

**三大限制**

- Cookie、LocalStorage 和 IndexDB 无法读取。
- DOM 无法获得。
- AJAX 请求不能发送，报跨域错误。

**IE 例外**

当涉及到同源策略时，Internet Explorer有两个主要的例外

- 授信范围（Trust Zones）：两个相互之间高度互信的域名，如公司域名（corporate domains），不遵守同源策略的限制。
- 端口：IE未将端口号加入到同源策略的组成部分之中，因此 http://company.com:81/index.html 和http://company.com/index.html  属于同源并且不受任何限制。

这些例外是非标准的，其它浏览器也未做出支持，但会助于开发基于window RT IE的应用程序

## 跨域

### 基础知识

跨域：根据同源，只要协议、域名、端口有任何一个不同，都被当作是不同的域。


	URL                      说明       是否允许通信
	http://www.a.com/a.js

	
	http://www.a.com/b.js     同一域名下   允许
	
	http://www.a.com/lab/a.js
	http://www.a.com/script/b.js 同一域名下不同文件夹 允许
	
	http://www.a.com:8000/a.js
	http://www.a.com:8000/b.js     同一域名，不同端口  不允许
	
	https://www.a.com/a.js
	https://www.a.com/b.js 同一域名，不同协议 不允许

	http://70.32.92.74/b.js 域名和域名对应ip 不允许
	
	http://script.a.com/b.js 主域相同，子域不同 不允许
	
	http://a.com/b.js 同一域名，不同二级域名（同上） 不允许（cookie这种情况下也不允许访问）
	
	http://www.cnblogs.com/a.js 不同域名 不允许

>对于端口和协议的不同，只能通过后台来解决。

### 解决方案

**1、document.domain**

浏览器都有一个同源策略，其限制之一就是第一种方法中我们说的不能通过ajax的方法去请求不同源中的文档。 它的第二个限制是浏览器中不同域的框架之间是不能进行js的交互操作的。

1）cookie

Cookie 是服务器写入浏览器的一小段信息，只有同源的网页才能共享。但是，两个网页一级域名相同，只是二级域名不同，浏览器允许通过设置document.domain共享 Cookie。

举例来说，A网页是http://w1.example.com/a.html，B网页是http://w2.example.com/b.html，那么只要设置相同的document.domain，两个网页就可以共享Cookie。

	document.domain = 'example.com';
现在，A网页通过脚本设置一个 Cookie。

	document.cookie = "test1=hello";
B网页就可以读到这个 Cookie。

	var allCookie = document.cookie;

另外，服务器也可以在设置Cookie的时候，指定Cookie的所属域名为一级域名，比如.example.com。

	Set-Cookie: key=value; domain=.example.com; path=/
这样的话，二级域名和三级域名不用做任何设置，都可以读取这个Cookie。

2) iframe

不同的框架之间是可以获取window对象的，但却无法获取相应的属性和方法。比如，有一个页面，它的地址是http://www.example.com/a.html ， 在这个页面里面有一个iframe，它的src是http://example.com/b.html, 很显然，这个页面与它里面的iframe框架是不同域的，所以我们是无法通过在页面中书写js代码来获取iframe中的东西的

如果两个窗口一级域名相同，只是二级域名不同，那么设置document.domain属性，就可以规避同源政策，拿到DOM。

	<script type="text/javascript">
	    function test(){
	        var iframe = document.getElementById('￼ifame');
	        var win = document.contentWindow;//可以获取到iframe里的window对象，但该window对象的属性和方法几乎是不可用的
	        var doc = win.document;//这里获取不到iframe里的document对象
	        var name = win.name;//这里同样获取不到window对象的name属性
	    }
	</script>
	<iframe id = "iframe" src="http://example.com/b.html" onload = "test()"></iframe>

1.在页面 http://www.example.com/a.html 中设置document.domain:

	<iframe id = "iframe" src="http://example.com/b.html" onload = "test()"></iframe>
	<script type="text/javascript">
	    document.domain = 'example.com';//设置成主域
	    function test(){
	        alert(document.getElementById('￼iframe').contentWindow);//contentWindow 可取得子窗口的 window 对象
	    }
	</script>
2.在页面 http://example.com/b.html 中也设置document.domain:

	<script type="text/javascript">
	    document.domain = 'example.com';//在iframe载入这个页面也设置document.domain，使之与主页面的document.domain相同
	</script>
修改document.domain的方法只适用于不同子域的框架间的交互。

>document.domain的设置是有限制的，我们只能把document.domain设置成自身或更高一级的父域，且主域必须相同。
>这种方法只适用于 Cookie 和 iframe 窗口，LocalStorage 和 IndexDB 无法通过这种方法，规避同源政策，而要使用下文介绍的PostMessage API。


**2、JSONP**

JSONP（JSON with Padding）是资料格式 JSON 的一种“使用模式”，可以让网页从别的网域要资料。

JSONP也叫填充式JSON，是应用JSON的一种新方法，只不过是被包含在函数调用中的JSON，例如：

	callback({"name","trigkit4"});
JSONP由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数，而数据就是传入回调函数中的JSON数据。

在js中，我们直接用XMLHttpRequest请求不同域上的数据时，是不可以的。但是，在页面上引入不同域上的js脚本文件却是可以的，jsonp正是利用这个特性来实现的。 例如：

	<script type="text/javascript">
	    function dosomething(jsondata){
	        //处理获得的json数据
	    }
	</script>
	<script src="http://example.com/data.php?callback=dosomething"></script>

如果你的页面使用jquery，那么通过它封装的方法就能很方便的来进行jsonp操作了。

	<script type="text/javascript">
	    $.getJSON('http://example.com/data.php?callback=?,function(jsondata)'){
	        //处理获得的json数据
	    });
	</script>
jquery会自动生成一个全局函数来替换callback=?中的问号，之后获取到数据后又会自动销毁，实际上就是起一个临时代理函数的作用。$.getJSON方法会自动判断是否跨域，不跨域的话，就调用普通的ajax方法；跨域的话，则会以异步加载js文件的形式来调用jsonp的回调函数

>JSONP的优点是：它不像XMLHttpRequest对象实现的Ajax请求那样受到同源策略的限制；它的兼容性更好，在更加古老的浏览器中都可以运行，不需要XMLHttpRequest或ActiveX的支持；并且在请求完毕后可以通过调用callback的方式回传结果。

>JSONP的缺点则是：它只支持GET请求而不支持POST等其它类型的HTTP请求；它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行JavaScript调用的问题。

**3、跨资源共享（CORS）**

更详细参考：[http://www.ruanyifeng.com/blog/2016/04/cors.html](http://www.ruanyifeng.com/blog/2016/04/cors.html)

CORS（Cross-Origin Resource Sharing）跨域资源共享，定义了必须在访问跨域资源时，浏览器与服务器应该如何沟通。CORS背后的基本思想就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功还是失败。

要跨域访问的接口地址：

	Access-Control-Allow-Origin： *

服务器端对于CORS的支持，主要就是通过设置Access-Control-Allow-Origin来进行的。如果浏览器检测到相应的设置，就可以允许Ajax进行跨域的访问。

**4、window.name**

浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。
父窗口先打开一个子窗口，载入一个不同源的网页，该网页将信息写入window.name属性。

	window.name = data;
接着，子窗口跳回一个与主窗口同域的网址。

	location = 'http://parent.url.com/xxx.html';
然后，主窗口就可以读取子窗口的window.name了。

	var data = document.getElementById('myFrame').contentWindow.name;

>这种方法的优点是，window.name容量很大，可以放置非常长的字符串；缺点是必须监听子窗口window.name属性的变化，影响网页性能。

**5、HTML5新特性**

**window.postMessage(message,targetOrigin)** 

此方法是html5新引进的特性，可以使用它来向其它的window对象发送消息，无论这个window对象是属于同源或不同源，目前IE8+、FireFox、Chrome、Opera等浏览器都已经支持window.postMessage方法。

举例来说，父窗口http://aaa.com向子窗口http://bbb.com发消息，调用postMessage方法就可以了。

	var popup = window.open('http://bbb.com', 'title');
	popup.postMessage('Hello World!', 'http://bbb.com');
postMessage方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin），即"协议 + 域名 + 端口"。也可以设为*，表示不限制域名，向所有窗口发送。
子窗口向父窗口发送消息的写法类似。

	window.opener.postMessage('Nice to see you', 'http://aaa.com');
父窗口和子窗口都可以通过message事件，监听对方的消息。

	window.addEventListener('message', function(e) {
	  console.log(e.data);
	},false);
message事件的事件对象event，提供以下三个属性:

- event.source：发送消息的窗口
- event.origin: 消息发向的网址
- event.data: 消息内容

**应用**

1）LocalStorage

通过window.postMessage，读写其他窗口的 LocalStorage 也成为了可能。

子窗口接收消息的代码如下。

	window.onmessage = function(e) {
	  if (e.origin !== 'http://bbb.com') return;
	  var payload = JSON.parse(e.data);
	  switch (payload.method) {
	    case 'set':
	      localStorage.setItem(payload.key, JSON.stringify(payload.data));
	      break;
	    case 'get':
	      var parent = window.parent;
	      var data = localStorage.getItem(payload.key);
	      parent.postMessage(data, 'http://aaa.com');
	      break;
	    case 'remove':
	      localStorage.removeItem(payload.key);
	      break;
	  }
	};
父窗口发送消息代码如下。

	var win = document.getElementsByTagName('iframe')[0].contentWindow;
	var obj = { name: 'Jack' };
	// 存入对象
	win.postMessage(JSON.stringify({key: 'storage', method: 'set', data: obj}), 'http://bbb.com');
	// 读取对象
	win.postMessage(JSON.stringify({key: 'storage', method: "get"}), "*");
	window.onmessage = function(e) {
	  if (e.origin != 'http://aaa.com') return;
	  // "Jack"
	  console.log(JSON.parse(e.data).name);
	};

**6、websocket**

WebSocket是一种通信协议，使用ws://（非加密）和wss://（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。


>至于一些跨域插件也都是基于上面的方式开发的：crossdomain.xml

>chrome的插件Allow-Control-Allow-Origin等。