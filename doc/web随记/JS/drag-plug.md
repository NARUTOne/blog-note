# 前端学习笔记の拖拽（二）插件篇

>引言：大家久等了，对不住了。最近经常出差，项目很赶啊，宝宝心很累/(ㄒoㄒ)/~~。
>参考：[http://www.cnblogs.com/lrzw32/p/4696655.html](http://www.cnblogs.com/lrzw32/p/4696655.html)

>源代码：[https://github.com/WZOnePiece/study-draggable](https://github.com/WZOnePiece/study-draggable)

##jquery 插件化##

起始插件化：

    ; (function($, window, undefined) {
        $.fn.drag  = function(options) {
            drag(this)
        }
  
    })(jquery, window, undefined);
    

其他的代码类似前篇拖拽基础篇[http://www.jianshu.com/p/feca77ec252e](http://www.jianshu.com/p/feca77ec252e)。只是需要将一些dom操作改为jquery操作。详细代码：

    ; (function($, window, undefined) {
      //拖拽元素
      function DragElement(node) {
        this.target = node;
        node.onselectstart = function() {
          return false;//防止文字选中
        }
      }
      DragElement.prototype = {
        constructor: DragElement,
        setXY: function(x, y) {
          this.x = parseInt(x) || 0;
          this.y = parseInt(y) || 0;
          return this;
        },
        setTargetCss: function(css) {
          $(this.target).css(css);
          return this;
        }
      }
    
      //鼠标
      function Mouse() {
        this.x = 0;
        this.y = 0;
      }
      Mouse.prototype.setXY = function(x, y) {
        this.x = parseInt(x);
        this.y = parseInt(y);
      }
    
      //拖拽配置
      var draggableConfig = {
        zIndex:10,
        dragElement: null,
        mouse: new Mouse()
      };
    
      var draggableStyle = {
        dragging: {
          cursor: 'move'
        },
        defaults: {
          cursor: 'auto'
        }
      }
    
      var $document = $(document);
    
      function drag($ele) {
        var $dragNode = $ele;
    
        $dragNode.on({
          'mousedown': function(event) {
            var dragElement = draggableConfig.dragElement = new DragElement($ele.get(0));
    
            draggableConfig.mouse.setXY(event.clientX, event.clientY);
            draggableConfig.dragElement.setXY(dragElement.target.style.left, dragElement.target.style.top)
              .setTargetCss({
                'zIndex': draggableConfig.zIndex ++,
                'position': 'relative'
              });
          },
          'mouseover': function() {
            $(this).css(draggableStyle.dragging);
          },
          'mouseout':function() {
            $(this).css(draggableStyle.defaults);
          }
        })
      }
    
      function move(event) {
        if(draggableConfig.dragElement) {
          var mouse = draggableConfig.mouse,
            dragElement = draggableConfig.dragElement;
          dragElement.setTargetCss({
            'left': parseInt(event.clientX - mouse.x + dragElement.x) + 'px',
            'top': parseInt(event.clientY - mouse.y + dragElement.y) + 'px'
          });
    
          $document.off('mousemove', move);
          setTimeout(function() {
            $document.on('mousemove', move);
          }, 25);
        }
      }
    
      $document.on({
        "mousemove": move,
        'mouseup': function() {
          draggableConfig.dragElement = null;
        }
      });
    
      $.fn.drag  = function(options) {
        drag(this)
      }
    
    })(jQuery, window, undefined);

拖拽引用：

      window.onload = function() {
        var dragObj = $("#drag");
        dragObj.drag();
    }

##jQuery拖拽插件 —— sortable##

###sortable.js###
>sortable.js:是一个独立的JS插件，不需要jquery，Sortable非常轻量，压缩后只有2KB，采用的是html5拖拽。[https://github.com/RubaXa/Sortable](https://github.com/RubaXa/Sortable)

>有关html5拖拽请期待 **拖拽三** O(∩_∩)O哈哈~。

简单例子：

    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <meta name="author" content="NARUTOne">
        <title>Sortable拖拽</title>
        <script type="text/javascript" src="./lib/Sortable.js"></script>
        <style media="screen">
          div {
            width: 800px;
            padding: 50px;
            border: 1px solid #ededed;
            margin: 100px auto;
          }
          div::after {
            content: '';
            display: table;
            clear: both;
          }
          p {
            width: 420px;
          }
          span {
            width: 120px;
            margin-right: 20px;
            margin-bottom: 10px;
            float: left;
            height:30px;
            /*border: 1px solid red;*/
            background-color: #26a4fe;
            color: #fff;
            font-weight: bold;
            text-align: center;
            line-height: 30px;
          }
        </style>
      </head>
      <body>
        <div class="">
          <p id="sortleft" class='item'>
            <span>1</span>
            <span>2</span>
            <span>3</span>
            <span>4</span>
            <span>5</span>
            <span>6</span>
            <span>7</span>
          </p>
        </div>
      </body>
      <script type="text/javascript">
        var el = document.getElementById('sortleft');
        new Sortable(el,{
          group: "name"
        });
      </script>
    </html>
    
>当然还有许多方法和配置，这就需要大神们自己看看文档了O(∩_∩)O哈哈~,小渣我就不说了。/(ㄒoㄒ)/~~

###jQueryUI——sortable###
>sortable.js也可以转换为jquery模式，需要引入一些文件，这里就不说了。现在说说jqueryUI这款插件，里面也是有类似sortable的拖拽插件的。

>官网：[http://jqueryui.com/](http://jqueryui.com/)

**简单案例：**

    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <meta name="author" content="NARUTOne">
      <title>jQuery UI 拖动（Draggable） + 排序（Sortable）</title>
      <link rel="stylesheet" href="./lib/jquery-ui.css">
      <script src="./lib/jquery-2.2.1.js"></script>
      <script src="./lib/jquery-ui.js"></script>
      <style>
      ul {
        list-style-type: none;
        margin: 0;
        padding: 0;
        margin-bottom: 10px;
       }
      li {
        margin: 5px;
        padding: 5px;
        width: 150px;
      }
      li svg {
          width:1em;
          height:1em;
        }
      </style>
      <script>
      $(function() {
        $( "#sortable" ).sortable({
          revert: true
        });
        $( "#draggable" ).draggable({
          connectToSortable: "#sortable",
          helper: "clone",
          revert: "invalid"
        });
        $( "ul, li" ).disableSelection();
      });
      </script>
    </head>
    <body>
     <svg style="display:none" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
        <symbol id="icon-delete" viewBox="0 0 25.128 31.87">
          <path d="M21.987,3.9H3.141C1.404,3.9,0,5.306,0,7.042v0.855h25.128V7.042c0-1.736-1.405-3.163-3.141-3.163   M16.5,1.91l0.463,2.624H8.165l0.463-2.5L16.5,1.971 M16.751,0H8.377C7.512,0,6.709,0.699,6.585,1.556l-0.61,3.518  c-0.12,0.855,0.49,1.554,1.352,1.554h10.472c0.863,0,1.474-0.699,1.354-1.556l-0.613-3.765C18.419,0.452,17.616,0,16.751,0   M22.512,9.922H2.616c-1.151,0-2.008,0.938-1.904,2.085l1.714,17.778c0.104,1.146,1.133,2.085,2.286,2.085h15.702  c1.153,0,2.182-0.938,2.284-2.085l1.717-17.778C24.518,10.86,23.663,9.922,22.512,9.922 M8.679,27.798H5.234l-1.047-14.04h4.491  V27.798z M14.556,27.798h-3.985v-14.04h3.985V27.798z M19.894,27.798h-3.245v-14.04h4.292L19.894,27.798z"></path>
        </symbol>
    </svg>
    <ul>
        <li id="draggable" class="ui-state-highlight">请拖拽我
            <svg ><use xlink:href="#icon-delete"></use></svg>
        </li>
    </ul>
    
    <ul id="sortable">
      <li class="ui-state-default">Item 1</li>
      <li class="ui-state-default">Item 2</li>
      <li class="ui-state-default">Item 3</li>
      <li class="ui-state-default">Item 4</li>
      <li class="ui-state-default">Item 5</li>
    </ul>
    
    </body>
    </html>

当然，在jqueryUI插件中还有许多有趣强大的功能，有兴趣的猿（媛）们可以看看。

>这篇文章是在比较忙里偷闲下写的，可能比较赖，没时间具体分析了，就直接上代码了。起始本人也是比较讨厌直接看代码的，这样就没有那种分享的意味了。哎，各位就先将就下吧，有问题，有bug可以评论下留言哈。(*^__^*) 嘻嘻……。

>接下来拖拽篇还有一篇HTML5下的拖拽，这篇我会好好的写的，好好分享下学习心得，好期待哈O(∩_∩)O哈！