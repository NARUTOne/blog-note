# JavaScript 事件这回事儿

> NARUTOne
> 
> 引言：js的事件机制在web开发中的出镜率是很高，但是要想给事件机制拍出高颜值的效果，还是很难的。本篇将会介绍javascript事件机制这回事儿。

>参考：[最详细的JavaScript和事件解读](http://www.codeceo.com/article/javascript-event-anay.html)
>
>[你若触发，我就处理——浅谈JavaScript的事件响应](http://blog.jobbole.com/51889/)

>[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Event#Properties)

**事件有哪些**

	//获取window事件
	var log = document.getElementById('log'),
    i = '', 
	out = [];
	for (i in window) {
	  if ( /^on/.test(i)) { out[out.length] = i; }
	}
	log.innerHTML = out.join(', ');

![](http://upload-images.jianshu.io/upload_images/2455352-86d5fab8de25eb5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以发现，事件家族成员还是很庞大的。

[标准事件属性列表](https://developer.mozilla.org/en-US/docs/Web/API/Event#Properties)

## 事件绑定监听

**html内联绑定**

	<a href='javascript:;' onclick="alert('你点击了这个a');">点击</a>

>显而易见，使用这种方法，JavaScript 代码与 HTML 代码耦合在了一起，不便于维护和开发。所以除非在必须使用的情况（例如统计链接点击数据）下，尽量避免使用这种方法。

**DOM属性绑定**
	
	element.onclick = function(event){
    	alert('你点击了这个按钮');
	};
>直接赋值给对应属性，如果你在后面代码中再次为 element 绑定一个回调函数，会覆盖掉之前回调函数的内容。

**事件监听**

	element.addEventListener(<event-name>, <callback>, <use-capture>);

表示在 element 这个对象上面添加一个事件监听器。

1、当监听到有 <event-name> 事件发生的时候。

2、调用 <callback> 这个回调函数。

3、至于 <use-capture> 这个参数，表示该事件监听是在“捕获”阶段中监听（设置为 true）还是在“冒泡”阶段中监听（设置为 false）。大部分设置在冒泡阶段。

	var btn = document.getElementsByTagName('button');
	btn[0].addEventListener('click', function() {
    	alert('你点击了这个按钮');
	}, false);

**移除监听**

	element.removeEventListener(<event-name>, <callback>, <use-capture>);

>需要注意的是，绑定事件时的回调函数不能是匿名函数，必须是一个声明的函数，因为解除事件绑定时需要传递这个回调函数的引用，才可以断开绑定。

	var fun = function() {
	    // function logic
	};
	
	element.addEventListener('click', fun, false);
	element.removeEventListener('click', fun, false);

##事件触发过程

事件绑定后，那事件发生时是如何确认发生对象和触发的呢，过程详看[w3c UI Events](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)

![](http://upload-images.jianshu.io/upload_images/2455352-4d835a2ea891a557.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**捕获阶段（Capture Phase）**

当我们在 DOM 树的某个节点发生了一些操作（例如单击、鼠标移动上去），就会有一个事件发射过去。这个事件从 Window 发出，不断经过下级节点直到目标节点。在到达目标节点之前的过程，就是捕获阶段（Capture Phase）。

所有经过的节点，都会触发这个事件。捕获阶段的任务就是建立这个事件传递路线，以便后面冒泡阶段顺着这条路线返回 Window。

监听某个在捕获阶段触发的事件，需要在事件监听函数传递第三个参数 true。

	element.addEventListener(<event-name>, <callback>, true);

**目标阶段（Target Phase）**

当事件跑啊跑，跑到了事件触发目标节点那里，最终在目标节点上触发这个事件，就是目标阶段。

需要注意的时，事件触发的目标总是最底层的节点。比如你点击一段文字，你以为你的事件目标节点在 div 上，但实际上触发在 <p>、<span> 等子节点上。

	document.addEventListener('click', function(e){
	    alert(e.target.tagName);
	}, false);

**冒泡阶段（Bubbling Phase）**

当事件达到目标节点之后，就会沿着原路返回，由于这个过程类似水泡从底部浮到顶部，所以称作冒泡阶段。

在实际使用中，你并不需要把事件监听函数准确绑定到最底层的节点也可以正常工作。比如在上例，你想为这个div 绑定单击时的回调函数，你无须为这个 div 下面的所有子节点全部绑定单击事件，只需要为 div 这一个节点绑定即可。因为发生它子节点的单击事件，都会冒泡上去，发生在 div 上面。


## 事件机制FQA

**为什么不用第三个参数 true（捕获）**

在使用 addEventListener 函数来监听事件时，第三个参数设置为 false，这样监听事件时只会监听冒泡阶段发生的事件。

这是因为 IE 浏览器不支持在捕获阶段监听事件，为了统一而设置的，毕竟 IE 浏览器的份额是不可忽略的。

**利用事件代理监听提升性能 （Event Delegate）**

因为事件有冒泡机制，所有子节点的事件都会顺着父级节点跑回去，所以我们可以通过监听父级节点来实现监听子节点的功能，这就是事件代理。

使用事件代理主要有两个优势：

- 减少事件绑定，提升性能。之前你需要绑定一堆子节点，而现在你只需要绑定一个父节点即可。减少了绑定事件监听函数的数量。
- 动态变化的 DOM 结构，仍然可以监听。当一个 DOM 动态创建之后，不会带有任何事件监听，除非你重新执行事件监听函数，而使用事件监听无须担忧这个问题。

``` markup
<ul id="resources">
  <li><a href="http://developer.mozilla.org">MDN</a></li>
  <li><a href="http://html5doctor.com">HTML5 Doctor</a></li>
  <li><a href="http://html5rocks.com">HTML5 Rocks</a></li>
  <li><a href="http://beta.theexpressiveweb.com/">Expressive Web</a></li>
  <li><a href="http://creativeJS.com/">CreativeJS</a></li>
</ul>
```

``` js
var resources = document.querySelector('#resources'),
    log = document.querySelector('#log');
 
resources.addEventListener('mouseover', showtarget, false);
 
function showtarget(ev) {
  var target = ev.target;
  if (target.tagName === 'A') {
    log.innerHTML = 'A link, with the href:' + target.href;
  }
  if (target.tagName === 'LI') {
    log.innerHTML = 'A list item';
  }
  if (target.tagName === 'UL') {
    log.innerHTML = 'The list itself';
  }
}
```
使用原生的方式实现事件代理，需要注意过滤非目标节点

	element.addEventListener('click', function(event) {
	    // 判断是否是 a 节点
	    if ( event.target.tagName == 'A' ) {
	        // a 的一些交互操作
	    }
	}, false);
	

**停止事件冒泡（stopPropagation）**

比较复杂的应用，由于事件监听比较复杂，可能会希望只监听发生在具体节点的事件。这个时候就需要停止事件冒泡。停止事件冒泡需要使用事件对象的 **stopPropagation** 方法

	element.addEventListener('click', function(event) {
	    event.stopPropagation();
	}, false);

**事件对象**
当一个事件被触发的时候，会创建一个事件对象（Event Object），这个对象里面包含了一些有用的属性或者方法。事件对象会作为第一个参数，传递给我们的毁掉函数。我们可以使用下面代码，在浏览器中打印出这个事件对象：[标准事件属性列表](https://developer.mozilla.org/en-US/docs/Web/API/Event#Properties)

	<button>打印 Event Object</button>
	
	<script>
	    var btn = document.getElementsByTagName('button');
	    btn[0].addEventListener('click', function(event) {
	        console.log(event);
	    }, false);
	</script>

![](http://upload-images.jianshu.io/upload_images/2455352-d6343e3875525bc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**默认事件禁止preventDefault**

这个方法可以禁止一切默认的行为，例如点击 a 标签时，会打开一个新页面，如果为 a 标签监听事件 click同时调用该方法，则不会打开新页面。

	<a href="www.baidu.com" id='jump'>是否跳转呢？</a>

	document.getElementById('jump').addEventListener('click', function(e) {
		alert('会跳转到www.baidu.com？？？');
		e.preventDefault();
	}, false)


## IE下的事件差异兼容

**IE 下绑定事件**

在 IE 下面绑定一个事件监听，在 IE9- 无法使用标准的 addEventListener 函数，而是使用自家的attachEvent，具体用法：

	element.attachEvent(<event-name>, <callback>);

其中 <event-name> 参数需要注意，它需要为事件名称添加 on 前缀，比如有个事件叫 click，标准事件监听函数监听 click，IE 这里需要监听 onclick。

另一个，它没有第三个参数，也就是说它只支持监听在冒泡阶段触发的事件，所以为了统一，在使用标准事件监听函数的时候，第三参数传递 false。

当然，这个方法在 IE9 已经被抛弃，在 IE11 已经被移除了，IE 也在慢慢变好。O(∩_∩)O~

** IE中的Event **

IE 中往回调函数中传递的事件对象与标准也有一些差异，你需要使用 window.event 来获取事件对象。所以你通常会写出下面代码来获取事件对象：

	event = event || window.event
此外还有一些事件属性有差别，比如比较常用的 event.target 属性，IE 中没有，而是使用 event.srcElement来代替。如果你的回调函数需要处理触发事件的节点，那么需要写：

	node = event.srcElement || event.target;

**用 JavaScript 模拟触发内置事件**

内置的事件也可以被 JavaScript 模拟触发，**dispatchEvent()**

	function simulateClick() {
	  var event = new MouseEvent('click', {
	    'view': window,
	    'bubbles': true,
	    'cancelable': true
	  });
	  var cb = document.getElementById('checkbox'); 
	  var canceled = !cb.dispatchEvent(event);
	  if (canceled) {
	    // A handler called preventDefault.
	    alert("canceled");
	  } else {
	    // None of the handlers called preventDefault.
	    alert("not canceled");
	  }
	}

## 自定义事件

内置的事件会由浏览器根据某些操作进行触发，自定义的事件就需要人工触发。dispatchEvent 函数就是用来触发某个事件

自定义事件的函数有 Event、CustomEvent 和 dispatchEvent

**使用 Event 构造函数**

	var event = new Event('build');

	// Listen for the event.
	elem.addEventListener('build', function (e) { ... }, false);
	
	// Dispatch the event.
	elem.dispatchEvent(event);

**CustomEvent创建**
CustomEvent 可以创建一个更高度自定义事件，还可以附带一些数据。

	var myEvent = new CustomEvent(eventname, options);

	var options = {
	    detail: {
	        ...
	    },
	    bubbles: true,
	    cancelable: false
	}
	
	element.dispatchEvent(customEvent);

detail 可以存放一些初始化的信息，可以在触发的时候调用。其他属性就是定义该事件是否具有冒泡等等功能。

在 element 上面触发 customEvent 这个事件。结合起来用就是：

	// add an appropriate event listener
	obj.addEventListener("cat", function(e) { process(e.detail) });
	
	// create and dispatch the event
	var event = new CustomEvent("cat", {"detail":{"hazcheeseburger":true}});
	obj.dispatchEvent(event);


使用自定义事件需要注意兼容性问题，而使用 jQuery 就简单多了：

	// 绑定自定义事件
	$(element).on('myCustomEvent', function(){});
	
	// 触发事件
	$(element).trigger('myCustomEvent');
此外，你还可以在触发自定义事件时传递更多参数信息：
	
	$( "p" ).on( "myCustomEvent", function( event, myName ) {
	  $( this ).text( myName + ", hi there!" );
	});
	$( "button" ).click(function () {
	  $( "p" ).trigger( "myCustomEvent", [ "John" ] );
	});

**事件解耦**

我们可以将一个整个的功能，分割成独立的小功能，每个小功能绑定一个事件，由一个“控制器”负责根据条件触发某个事件。这样，在外面触发这个事件，也可以调用对应功能，使其更加灵活。

![](http://upload-images.jianshu.io/upload_images/2455352-312832e5951893af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里推荐 cowboy 开发的 [Tiny Pub Sub](https://github.com/cowboy/jquery-tiny-pubsub)，通过 jQuery 实现

	(function($) {
	
	  var o = $({});
	
	  $.subscribe = function() {
	    o.on.apply(o, arguments);
	  };
	
	  $.unsubscribe = function() {
	    o.off.apply(o, arguments);
	  };
	
	  $.publish = function() {
	    o.trigger.apply(o, arguments);
	  };
	
	}(jQuery));


## 媒体事件

[video](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video) 和 [audio](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Using_HTML5_audio_and_video)这俩很潮的玩意也有一大堆事件供我们使用。比如有趣的time事件，它可以告诉我们这首歌或电影的已播放时长。

##其他设备事件

我们知道，浏览器提供了与鼠标键盘的交互，但这还远远不够满足我们更多的硬件交互需求。比如检测手机或平板电脑倾斜度的Device orientation和 [touch events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)。the Gamepad API让我们可以在浏览器中做游戏控制；postMessage让我们可以在浏览器各窗口之间进行跨域消息传递；pageVisibility让我们可以得知浏览器中当前标签页可见状态。甚至当window的history对象有操作时也能监听的到。查看window对象的事件列表，有的可能已经被实现了，还有更多的在谋划中……

嘛，不管浏览器是否会支持，最终都是要支持的嘛，这些是刚需。我们只要默默等待就可以了，骚年，向着夕阳奔跑吧！=v=


##总结

>**绑定、捕获、冒泡、委托及浏览器兼容。**

1、绑定三种方式：

①onEle = fun模式 ：

②addEventListener('ele', fun, bool)模式：

③attachEvent('onEle', fun, bool)模式：

2、默认事件行为阻止（一般）：

①on模式：

    ele.onclick = function() {
        ……                         //你的代码
        return false;              //通过返回false值阻止默认事件行为
    };
    
②addEventListener()模式：

    element.addEventListener("click", function(e){
        var event = e || window.event;
        ……
        event.preventDefault( );      //阻止默认事件
    },false);
    
③attachEvent()模式 **IE**

    element.attachEvent("onclick", function(e){
        var event = e || window.event;
        ……
        event.returnValue = false;       //阻止默认事件
    },false);
    
    
3、兼容绑定、解绑

    // 事件绑定
    function addEvent(element, eType, handle, bol) {
        if(element.addEventListener){           //如果支持addEventListener
            element.addEventListener(eType, handle, bol);
        }else if(element.attachEvent){          //如果支持attachEvent
            element.attachEvent("on"+eType, handle);
        }else{                                  //否则使用兼容的onclick绑定
            element["on"+eType] = handle;
        }
    }
    
    // 事件解绑
    function removeEvent(element, eType, handle, bol) {
        if(element.addEventListener){
            element.removeEventListener(eType, handle, bol);
        }else if(element.attachEvent){
            element.detachEvent("on"+eType, handle);
        }else{
            element["on"+eType] = null;
        }
    }
    
 4、冒泡
    
    // 阻止事件的进一步传播，包括（冒泡，捕获），无参数
     event.stopPropagation( );                
     
    event.cancelBubble = true;  // true 为阻止冒泡，IE
    
    
5、事件委托：利用事件冒泡的特性，将里层的事件委托给外层事件，根据event对象的属性进行事件委托，改善性能。  
使用事件委托能够避免对特定的每个节点添加事件监听器；事件监听器是被添加到它们的父元素上。事件监听器会分析从子元素冒泡上来的事件，找到是哪个子元素的事件。