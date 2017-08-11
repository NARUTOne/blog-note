# 前端学习笔记の拖拽（一）原生基础篇

#前端学习笔记の拖拽（一）#
>源码地址：https://github.com/WZOnePiece/study-draggable
>引言：9月开始的半个月，由于项目和一些朋友的询问，我接触了关于前端拖拽的一些问题。不过本人由于只是一个前端菜鸟，接触前端时间满打满算，从0到现在也就一年左右，对于一些知识了解的还不是很全面和深刻。所以，我就打算好好学习下前端拖拽得一些相关知识。这就是这篇“史诗级的锦绣文章”的诞生记O(∩_∩)O。
>借鉴：http://www.cnblogs.com/lrzw32/p/4696655.html

##js拖拽插件的原理##

常见的拖拽操作是什么样的呢？整过过程大概有下面几个步骤：

　　1、用鼠标点击被拖拽的元素

　　2、按住鼠标不放，移动鼠标

　　3、拖拽元素到一定位置，放开鼠标

这里的过程涉及到三个dom事件：onmousedown,onmousemove,onmouseup。所以拖拽的基本思路就是：

　　1、用鼠标点击被拖拽的元素触发onmousedown

　　　　（1）设置当前元素的可拖拽为true，表示可以拖拽

　　　　（2）记录当前鼠标的坐标x,y

　　　　（3）记录当前元素的坐标x,y

　　2、移动鼠标触发onmousemove

　　　　（1）判断元素是否可拖拽，如果是则进入步骤2，否则直接返回

　　　　（2）如果元素可拖拽，则设置元素的坐标

　　　　　　元素的x坐标 = 鼠标移动的横向距离+元素本来的x坐标 = 鼠标现在的x坐标 - 鼠标之前的x坐标 + 元素本来的x坐标

　　　　　　元素的y坐标 = 鼠标移动的横向距离+元素本来的y坐标 = 鼠标现在的y坐标 - 鼠标之前的y坐标 + 元素本来的y坐标

　　3、放开鼠标触发onmouseup

　　　　（1）将鼠标的可拖拽状态设置成false

##基本效果##

在实现基本的效果之前，有几点需要说明的：

　　1、元素想要被拖动，它的postion属性一定要是relative或absolute

　　2、通过event.clientX和event.clientY获取鼠标的坐标

　　3、onmousemove是绑定在document元素上而不是拖拽元素本身，这样能解决快速拖动造成的延迟或停止移动的问题

代码如下：

    var dragObj = document.getElementById("test");
    dragObj.style.left = "0px";
    dragObj.style.top = "0px";

    var mouseX, mouseY, objX, objY;
    var dragging = false;

    dragObj.onmousedown = function (event) {
        event = event || window.event;

        dragging = true;
        dragObj.style.position = "relative";

        mouseX = event.clientX;
        mouseY = event.clientY;
        objX = parseInt(dragObj.style.left);
        objY = parseInt(dragObj.style.top);
    }

    document.onmousemove = function (event) {
        event = event || window.event;
        if (dragging) {

            dragObj.style.left = parseInt(event.clientX - mouseX + objX) + "px";
            dragObj.style.top = parseInt(event.clientY - mouseY + objY) + "px";
        }

    }

    document.onmouseup = function () {
        dragging = false;
    }

##代码抽象和优化##

上面的代码要做成插件，要将其抽象出来，基本思路如下：

    ; (function (window, undefined) {            

            function Drag(ele) {}

            window.Drag = Drag;
        })(window, undefined);

用自执行匿名函数将代码包起来，内部定义Drag方法并暴露到全局中，直接调用Drag，传入被拖拽的元素。

>前面加入； 避免一些不必要错误。

首先对一些常用的方法进行简单的封装：

    ; (function (window, undefined) {
            var dom = {
                //绑定事件
                on: function (node, eventName, handler) {
                    if (node.addEventListener) {
                        node.addEventListener(eventName, handler);
                    }
                    else {
                        node.attachEvent("on" + eventName, handler);
                    }
                },
                //获取元素的样式
                getStyle: function (node, styleName) {
                    var realStyle = null;
                    if (window.getComputedStyle) {
                        realStyle = window.getComputedStyle(node, null)[styleName];
                    }
                    else if (node.currentStyle) {
                        realStyle = node.currentStyle[styleName];
                    }
                    return realStyle;
                },
                //获取设置元素的样式
                setCss: function (node, css) {
                    for (var key in css) {
                        node.style[key] = css[key];
                    }
                }
            };
          
            window.Drag = Drag;
        })(window, undefined);

在一个拖拽操作中，存在着两个对象：被拖拽的对象和鼠标对象，我们定义了下面的两个对象以及它们对应的操作：

首先的拖拽对象，它包含一个元素节点和拖拽之前的坐标x和y：

    function DragElement(node) {
        this.node = node;//被拖拽的元素节点
        this.x = 0;//拖拽之前的x坐标
        this.y = 0;//拖拽之前的y坐标
    }
    DragElement.prototype = {
        constructor: DragElement,
        init: function () {                    
            this.setEleCss({
                "left": dom.getStyle(node, "left"),
                "top": dom.getStyle(node, "top")
            })
            .setXY(node.style.left, node.style.top);
        },
        //设置当前的坐标
        setXY: function (x, y) {
            this.x = parseInt(x) || 0;
            this.y = parseInt(y) || 0;
            return this;
        },
        //设置元素节点的样式
        setEleCss: function (css) {
            dom.setCss(this.node, css);
            return this;
        }
    }

还有一个对象是鼠标，它主要包含x坐标和y坐标：

    function Mouse() {
            this.x = 0;
            this.y = 0;
        }
    Mouse.prototype.setXY = function (x, y) {
        this.x = parseInt(x);
        this.y = parseInt(y);
    }
    

这是在拖拽操作中定义的两个对象。

 

如果一个页面可以有多个拖拽元素，那应该注意什么：

1、每个元素对应一个拖拽对象实例

2、每个页面只能有一个正在拖拽中的元素

为此，我们定义了唯一一个对象用来保存相关的配置：

     var draggableConfig = {
         zIndex: 1,
       draggingObj: null,
         mouse: new Mouse()
    };

这个对象中有三个属性：

（1）zIndex：用来赋值给拖拽对象的zIndex属性，有多个拖拽对象时，当两个拖拽对象重叠时，会造成当前拖拽对象有可能被挡住，通过设置zIndex使其显示在最顶层

（2）draggingObj：用来保存正在拖拽的对象，在这里去掉了前面的用来判断是否可拖拽的变量，通过draggingObj来判断当前是否可以拖拽以及获取相应的拖拽对象

（3）mouse：唯一的鼠标对象，用来保存当前鼠标的坐标等信息

最后是绑定onmousedown，onmouseover，onmouseout事件，整合上面的代码如下：

    /*
      author: NARUTOne,wz;
      creatTime: 2016/9/20;
      developing: 可以扩展：添加父元素限制；多个同级元素拖拽顺序排列等。
    */
    
    ; (function(window, undefined) {
      // 拖拽对象的事件绑定；样式获取、设置
      var dom = {
        on: function(node, eventName, handler) {
          if(node.addEventListener) {
            console.log(node);
            node.addEventListener(eventName, handler, false);
          }
          else {//IE
            node.attachEvent("on" + eventName, handler);
          }
        },
        getStyle: function(node, styleName) {
          var realStyle = null;
          if(window.getComputedStyle) {
            realStyle = window.getComputedStyle(node,null)[styleName];
          }
          else if(node.currentStyle) {//IE
            realStyle = node.currentStyle[styleName];
          }
          else if(node.style) {
            realStyle = node.style[styleName];
          }
          return realStyle;
        },
        setCss: function(node, css) {
          for(var key in css) {
            node.style[key] = css[key];
          }
        }
      };
    
    // 拖拽对象定义
      function DragElement(node) {
        this.node = node;
        this.x = 0;
        this.y = 0;
      }
    
      DragElement.prototype = {
        constructor: DragElement,
        init: function() {
          this.setEleCss({
            "left": dom.getStyle(node, "left"),
            "top": dom.getStyle(node, "top")
          })
          .setXY(node.style.left, node.style.top)
        },
        // 设置当前坐标
        setXY: function(x, y) {
          this.x = parseInt(x) || 0;
          this.y = parseInt(y) || 0;
          return this;
        },
        // 设置元素节点的样式
        setEleCss: function(css) {
          dom.setCss(this.node, css);
          return this;
        }
      }
    
      // 鼠标对象
      function Mouse() {
        this.x = 0;
        this.y = 0;
      }
      Mouse.prototype.setXY = function(x, y) {
        this.x = parseInt(x);
        this.y = parseInt(y);
      }
    
      // 拖拽配置
      var draggableConfig = {
        zIndex: 1,
        draggingObj: null,
        mouse: new Mouse(),
        parent:document,
        dragEle:null
      }
    
      // 拖拽实现
      function Drag(ele,parent) {
        this.ele = ele;
    
        function mouseDown(event) {
          var elem = this;
    
          draggableConfig.mouse.setXY(event.clientX, event.clientY);
    
          draggableConfig.draggingObj = new DragElement(elem);
    
          draggableConfig.draggingObj
            .setXY(elem.style.left, elem.style.top)
            .setEleCss({
              "zIndex": draggableConfig.zIndex ++,
              "position": "absolute"
            });
        }
        draggableConfig.parent = parent;
        draggableConfig.dragEle = ele;
        ele.onselectstart = function() {
          // 防止拖拽对象内文字被选中
          return false;
        }
        dom.on(ele, "mousedown", mouseDown);
      }
    
      dom.on(draggableConfig.parent, "mousemove", function(e) {
        if(draggableConfig.draggingObj) {
          var mouse = draggableConfig.mouse,
            draggingObj = draggableConfig.draggingObj;
            // 这里可以添加限制，拖拽不脱离父元素
          var p_height = draggableConfig.parent.offsetHeight;
          var p_width = draggableConfig.parent.offsetWidth;
          var c_height = draggableConfig.dragEle.offsetHeight;
          var c_width = draggableConfig.dragEle.offsetWidth;
          var left, top = 0;
    
          if(event.clientX - mouse.x + draggingObj.x < 0) {
            left = 0;
          }
          else if((event.clientX - mouse.x + draggingObj.x) > (p_width - c_width)) {
            left = p_width - c_width;
          }
          else {
            left = event.clientX - mouse.x + draggingObj.x;
          }
    
          if(event.clientY - mouse.y + draggingObj.y < 0) {
            top = 0;
          }
          else if((event.clientY - mouse.y + draggingObj.y) > (p_height - c_height)) {
            top = p_height - c_height;
          }
          else {
            top = event.clientY - mouse.y + draggingObj.y;
          }
    
          draggingObj.setEleCss({
            "left": parseInt(left) + "px",
            "top": parseInt(top) + "px"
          })
        }
      })
    
      dom.on(draggableConfig.parent, "mouseup", function(event) {
        draggableConfig.draggingObj = null;
      })
    
      window.Drag = Drag;
    
    })(window,undefined);

调用方法：Drag(document.getElementById("obj"));

> **注意点：**为了防止选中拖拽元素中的文字，通过onselectstart事件处理程序return false来处理这个问题。或者css3中的user-select:none;  
> srcElement是IE下的属性，target是Firefox下的属性，Chrome浏览器同时有这两个属性。  
> 当然这个也是可以按照需要进行定制化扩展等。

##扩展：有效拖拽##

有时，会遇到一些拖拽需求，只能拖拽需要拖拽模块的部分区域，这时怎样实现呢？  

**思路：**首先优化拖拽元素对象如下，增加一个目标元素target，表示被拖拽对象，在上图的登录框中，就是整个登录窗口。  
被记录和设置坐标的拖拽元素就是这个目标元素，但是它并不是整个部分都是拖拽的有效部分。我们在html结构中为拖拽的有效区域添加类draggable表示有效拖拽区域：

    <div id="drag">
        <h3 class='drag-title'>拖拽主体</h3>
        <div class="drag-body">
            内容主体
        </div>
    </div>
    
js部分：

    function Drag(ele,parent) {
        this.ele = ele;
        var drag = ele.getElementsByClassName('drag-title')[0] || ele
        function mouseDown(event) {
          var elem = this ;
          draggableConfig.mouse.setXY(event.clientX, event.clientY);
          draggableConfig.draggingObj = new DragElement(ele);
          draggableConfig.draggingObj
            .setXY(ele.style.left, ele.style.top)
            .setEleCss({
              "zIndex": draggableConfig.zIndex ++,
              "position": "absolute"
            });
        }
        draggableConfig.parent = parent;
        draggableConfig.dragEle = ele;
        ele.onselectstart = function() {
          // 防止拖拽对象内文字被选中
          return false;
        }
        dom.on(drag, "mousedown", mouseDown);
      }
      
##性能优化##
由于onmousemove在一直调用，会造成一些性能问题，我们可以通过setTimout来延迟绑定onmousemove事件，改进move函数如下：

     setTimeout(function () {
        dom.on(document, "mousemove", function(e){
        
        });
     }, 25);

>**接下来的拖拽（二）将会去学习下H5中拖拽，敬请下回分解哈，O(∩_∩)O哈哈~**