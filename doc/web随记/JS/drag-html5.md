# 前端学习随记の拖拽（三）HTML5篇

>HTML5篇是拖拽学习的最后一篇了，O(∩_∩)O哈哈~。！

>参考传送门：[http://www.cnblogs.com/fsjohnhuang/p/3961066.html#t3](http://www.cnblogs.com/fsjohnhuang/p/3961066.html#t3)，这位猿还是很有耐性的，介绍的很详细啊，下了大功夫啊(*^__^*) 嘻嘻……。

>源代码传送门：[https://github.com/WZOnePiece/study-draggable](https://github.com/WZOnePiece/study-draggable)

HTML5之前，要实现网页元素的拖放操作，需要依靠mousedown、mousemove、mouseup等API，通过大量的JS代码来实现；HTML5中引入了直接支持拖放操作的API，大大简化了网页元素的拖放操作编程难度，并且这些API除了支持浏览器内部元素的拖放外，同时支持浏览器和其它应用程序之间的数据互相拖动。

###相关API###

1、DataTransfer 对象：拖拽对象用来传递的媒介，使用一般为Event.dataTransfer。内部方法：

- setData(type, data)：设置被拖数据的数据类型和值;
- getData(type)：获取拖动数据值；
- clearData(type)：数据清除。

2、 draggable 属性：就是标签元素要设置draggable=true，否则不会有效果，例如：

    <div title="拖拽我" draggable="true">列表1</div>

3、ondrag：表示鼠标拖拽的过程,类似于onmousemove。作用在源元素上。

4、ondragstart 事件：当拖拽元素开始被拖拽的时候触发的事件，此事件作用在被拖拽元素上（源元素）。

5、ondragenter 事件：当拖曳元素进入目标元素的时候触发的事件，此事件作用在目标元素上。

6、ondragover 事件：拖拽元素在目标元素上移动的时候触发的事件，此事件作用在目标元素上。

7、ondrop 事件：被拖拽的元素在目标元素上同时鼠标放开触发的事件，此事件作用在目标元素上。

8、ondragend 事件：当拖拽完成后触发的事件，此事件作用在被拖曳元素上。

9、dragleave事件：源元素离开目标元素,触发该事件。

10、Event.preventDefault() 方法：阻止默认的些事件方法等执行。在ondragover中一定要执行preventDefault()，否则ondrop事件不会被触发。另外，如果是从其他应用软件或是文件中拖东西进来，尤其是图片的时候，默认的动作是显示这个图片或是相关信息，并不是真的执行drop。此时需要在目标元素的ondragover事件中把它直接干掉。

11、Event.effectAllowed 属性：就是拖拽的效果，鼠标指针类型，仅能在dragover中设置。

###简单例子：###

    <!DOCTYPE html>
    <html>
     <head>
      <title>实现HTML5中的拖拽效果</title>
      <meta charset="utf-8"/>
      <meta name='author' content='NARUTOne'>
      <style>
        #d1 {
            width : 300px;
            height : 300px;
            border : solid 1px black;
            float : left;
        }
        #d2 {
            width : 300px;
            height : 300px;
            border : solid 1px black;
            float : right;
        }
      </style>
     </head>
    
     <body>
        <!-- 用于显示源元素(DATA文件夹) -->
        <div id="d1">
            <p>源元素</p>
            <!-- 将该图片作为源元素 -->
            [站外图片上传中……(3)]</img>
        </div>
        <!-- 用于显示目标元素(PROJECT文件夹) -->
        <div id="d2">
            <p>目标元素</p>
        </div>
      <script>
          // 1. 获取源元素 - 图片元素
          var ele1 = document.getElementById("myimg");
          var d1 = document.getElementById("d1");
          // 2. 获取目标元素 - id为d2的div元素
          var d2 = document.getElementById("d2");
          // 3. 源元素事件 - dragstart事件
          ele1.addEventListener("dragstart",function(event){
              // a. 获取到源元素使用的数据 - src属性值
              var mysrc = ele1.src;
              console.log(mysrc);
              // b. 将数据设置到dataTransfer对象中
              event.dataTransfer.setData("text",mysrc);//text,指数据类型type
          });
    
          // 4. 目标元素事件 - drop事件;自定义处理拖放过程
          d2.addEventListener("drop",function(event){
              // a. 阻止页面的默认行为
              event.preventDefault();
              // b. 从dataTransfer对象得到数据
              var mysrc = event.dataTransfer.getData("text");//返回指定type的数据
              // c. 创建<img>元素,设置一些属性
              var myimg = document.createElement("img");
              myimg.src = mysrc;
          myimg.id = 'myimg';
              myimg.width = "200";
    
              // f. 将源元素从页面中删除
          var removeElem = document.getElementById('myimg');
              d1.removeChild(removeElem);
          // d. 将<img>元素添加到id为d2的div元素中
          d2.appendChild(myimg);
          // e. 清除dataTransfer对象中的数据内容
          event.dataTransfer.clearData("text");
          });
          /*
           * dragover和dragleave事件
           * * dragover - 源元素到达目标元素
           * * dragleave - 源元素离开目标元素
           * * 上述两个事件组合效果 - 拖拽效果
           * drop事件
           * * 完成逻辑
           */
          d2.addEventListener("dragover",function(){
              event.preventDefault();
          });
          d2.addEventListener("dragleave",function(){
              event.preventDefault();
          });
    
    //d1作为目标元素
        d1.addEventListener("drop",function(event){
          event.preventDefault();
          // b. 从dataTransfer对象得到数据
          var mysrc = event.dataTransfer.getData("text");
          // c. 创建<img>元素,设置一些属性
          var myimg = document.createElement("img");
          myimg.src = mysrc;
          myimg.width = "200";
          myimg.id = 'myimg';
          // f. 将源元素从页面中删除
    
          var removeElem = document.getElementById('myimg');
          d2.removeChild(removeElem);
    
          d1.appendChild(myimg);
          event.dataTransfer.clearData("text");
        });
    
        d1.addEventListener("dragover",function(){
          event.preventDefault();
        });
        d1.addEventListener("dragleave",function(){
          event.preventDefault();
        });
          /*
          setDragImage()用于在拖放过程中，修改鼠标指针所指向的图像
              event.dataTransfer.setDragImage(image,x,y);
              event.target;返回触发事件的元素（事件的目标节点）
          */
      </script>
     </body>
    </html>

效果图，但是比较丑(✿◡‿◡)：
![](http://upload-images.jianshu.io/upload_images/2455352-49df33c7d7098554.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/2455352-034d54b35f3a90fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>HTML5是存在兼容性问题的，则drag and drop (DnD)也是有兼容性问题的。例：IE6-7在dataTransfer对象存在兼容性问题。具体的兼容问题，还是需要具体问题具体解决了！
>如果猿（媛）们想要了解更加具体的HTML5拖拽问题和技术，我建议可以看看文章开头的参考链接里面的文章，写的还是很好的。我本来想多借鉴下，写写文章什么的，想想还是算了(*^__^*) 嘻嘻……。毕竟人家找资料总结也是不容易啊。