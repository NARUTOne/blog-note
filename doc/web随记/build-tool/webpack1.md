# webpackの学习随记（一）

![](http://upload-images.jianshu.io/upload_images/2455352-f033822f7020a43b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#webpack #
>https://fakefish.github.io/react-webpack-cookbook/  
>http://webpack.github.io/
>https://segmentfault.com/a/1190000005742122

![](http://upload-images.jianshu.io/upload_images/2455352-cfd5b52ac3a6619a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### webpack简介 ###
事实上它是一个打包工具，而不是像RequireJS或SeaJS这样的模块加载器，通过使用Webpack，能够像Node.js一样处理依赖关系，然后解析出模块之间的依赖，将代码打包。

Webpack 只是一个模块合并器！也就是说你可以设置他去加载任何你写的匹配，只要有一个加载器。

webpack.config.js 简单例子

	var webpack = require('webpack');
	var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('common.js');
	
	module.exports = {
	    entry: {
	        entry1: './entry/entry1.js',
	        entry2: './entry/entry2.js'
	    },
	    output: {
	        path: __dirname,
	        filename: '[name].entry.js'
	    },
	    resolve: {
	        extensions: ['', '.js', '.jsx']
	    },
	    module: {
	        loaders: [{
	            test: /\.js$/,
	            loader: 'babel-loader'
	        }, {
	            test: /\.jsx$/,
	            loader: 'babel-loader!jsx-loader?harmony'
	        }]
	    },
	    plugins: [commonsPlugin]
	};


- entry：指定打包的入口文件，每有一个键值对，就是一个入口文件
output：配置打包结果，path定义了输出的文件夹，filename则定义了打包结果文件的名称，filename里面的[name]会由entry中的键（这里是entry1和entry2）替换。
- resolve：定义了解析模块路径时的配置，常用的就是extensions，可以用来指定模块的后缀，这样在引入模块时就不需要写后缀了，会自动补全。
- module：定义了对模块的处理逻辑，这里可以用loaders定义了一系列的加载器，以及一些正则。当需要加载的文件匹配test的正则时，就会调用后面的loader对文件进行处理，这正是webpack强大的原因。比如这里定义了凡是.js结尾的文件都是用babel-loader做处理，而.jsx结尾的文件会先经过jsx-loader处理，然后经过babel-loader处理。当然这些loader也需要通过npm install安装。
- plugins: 这里定义了需要使用的插件，比如commonsPlugin在打包多个入口文件时会提取出公用的部分，生成common.js。  

当然Webpack还有很多其他的配置，具体可以参照[http://webpack.github.io/docs/](http://webpack.github.io/docs/ "配置文档")

## 开始webpack之旅 ##

### 配置安装 ###
1、node和npm：安装到最新，官网[https://nodejs.org/en/](https://nodejs.org/en/)

2、创建package.json：问题随便，之后可以改
	
	npm init

3、安装webpack

	//安装
	npm i webpack --save-dev
	//运行
	node_modules/.bin/webpack

4、配置webpack.config.js

	var path = require('path');

	module.exports = {
	    entry: path.resolve(__dirname, 'app/main.js'),
	    output: {
	        path: path.resolve(__dirname, 'build'),
	        filename: 'bundle.js',
	    },
	};
5、打包配置，package.json

	"scripts": {
    	"build": "webpack"
	}

运行 

	npm run build 

打包 build.js 

>注：npm 会自动找到webpack；如果需要考虑未知环境，可以采用gulp-webpack解决方案。


### 工作流 ###
**利用webpack-dev-server，创建服务，替换npm run build。**

1、下载安装

	npm i webpack-dev-server --save

2、package.json配置

	{
	  "scripts": {
	    "build": "webpack",
	    "dev": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build"
	  }
	}

- webpack-dev-server - 在 localhost:8080 建立一个 Web 服务器。
- --devtool eval - 为你的代码创建源地址。当有任何报错的时候可以让你更加精确地定位到文件和行号。
- --progress - 显示合并代码进度。
- --colors - Yay，命令行中显示颜色！
- --content-base build - 指向设置的输出目录。

3、测试运行  

	npm run dev

[http://localhost:8080](http://localhost:8080)

4、自动刷新  
当运行 webpack-dev-server 的时候，它会监听你的文件修改。当项目重新合并之后，会通知浏览器刷新。

①index.html需要和build.js 处在同一目录下，引入build.js  
②在webpack.config.js中增加一个刷新入口点

	entry: [
      'webpack/hot/dev-server',
      'webpack-dev-server/client?http://localhost:8080',
      path.resolve(__dirname, 'app/main.js')
    ],

### 文件引入 ###

1、模块引入方式

ES6 模块

	import MyModule from './MyModule.js';
CommonJS

	var MyModule = require('./MyModule.js');
AMD

	define(['./MyModule.js'], function (MyModule) {
	
	});

2、路径

相对路径是相对当前目录，绝对路径是相对入口文件。

	// ES6 相对路径
	import utils from './../utils.js';
	
	// ES6 绝对路径
	import utils from '/utils.js';
	
	// CommonJS 相对路径
	var utils = require('./../utils.js');
	
	// CommonJS 绝对路径
	var utils = require('/utils.js');

>注：文件后缀，你不需要去特意去使用 .js，但是他能够更让你更清楚你正引入的档案。因為你可能有一些 .js 文件和一些 .jsx 文件，甚至一些图片和 css 可以用 Webpack 來引入。加入文件后缀，可以让你清楚地区分你引入的是 node_modules 或特定档案还是一般文件档案。

## ReactJS ##

### 配置ReactJS ###
![](http://upload-images.jianshu.io/upload_images/2455352-a525e0080c99633d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1、下载安装

	//安装react
	npm install react --save
	npm install react-dom --save
	//安装babel-loader  
	npm install babel-loader --save-dev
	npm install babel-core --save-dev
	npm install babel-preset-es2015 --save-dev  //支持ES2015
	npm install babel-preset-react --save-dev //支持jsx
	npm install babel-preset-stage-0 --save-dev //支持ES7
	//但是还不够
	npm install babel-polyfill --save
	/*
	Polyfilla是一个英国产品,在美国称之为Spackling Paste(译者注:刮墙的,在中国称为腻子).记住这一点就行:把旧的浏览器想象成为一面有了裂缝的墙.这些[polyfills]会帮助我们把这面墙的裂缝抹平,还我们一个更好的光滑的墙壁(浏览器)
	*/
	npm install babel-runtime  --save
	npm install babel-plugin-transform-runtime --save-dev
	/*减少打包的时候重复代码，以上要注意是放在dev还是非dev上！*/
		
>注：babel 6.0+；ES6及jsx编译

2、webpack.config.js 配置

	//在入口添加polyfill
	entry[
	'babel-polyfill',
	'webpack/hot/dev-server',
	'webpack-dev-server/client?http://localhost:8080',
	path.resolve(__dirname,'app/main.js')
	]
	//在webpack.config.js的module.exports = {}里面增加
	module:{
	loaders:[{
	    'loader':'babel-loader',
	    exclude:[
	        //在node_modules的文件不被babel理会
	        path.resolve(__dirname,'node_modules'),
	    ]，
	    include:[
	        //指定app这个文件里面的采用babel
	        path.resolve(__dirname,'app'),
	    ],
	    test：/\.jsx?$/,
	    query:{
	        plugins:['transform-runtime'],
	        presets:['es2015','stage-0','react']
	    }
	}]
	}

3、运行打包
	
	//打包
	npm run build 
	//运行
	npm run dev

### 优化重合并 ###

在开发环境中，最理想的是编译最多 200 到 800 毫秒的速度，取决于你在开发的应用。

为了不让 Webpack 去遍历 React JS 及其所有依赖，你可以在开发中重写它的行为。

***webpack.config.js***

	var path = require('path');
	var node_modules = path.resolve(__dirname, 'node_modules');
	var pathToReact = path.resolve(node_modules, 'react/dist/react.min.js');
	
	config = {
	    entry: ['webpack/hot/dev-server', path.resolve(__dirname, 'app/main.js')],
	    resolve: {
	        alias: {
	          'react': pathToReact
	        }
	    },
	    output: {
	        path: path.resolve(__dirname, 'build'),
	        filename: 'bundle.js',
	    },
	    module: {
	        loaders: [{
	            test: /\.jsx?$/,
	            loader: 'babel'
	        }],
	        noParse: [pathToReact]
	    }
	};
	
	module.exports = config;

- 每当 "react" 在代码中被引入，它会使用压缩后的 React JS 文件，而不是到 node_modules 中找。
- 每当 Webpack 尝试去解析那个压缩后的文件，我们阻止它，因为这不必要。

>注意！这个是设置一个压缩和发布的 React 版本，结果你可能会失去 propTypes 基础类型检查！

### Flow ###

进行类型检测

flowcheck-loader

## CSS、Fonts和Images加载 ##
Webpack允许像加载任何代码一样加载 CSS，你可以为每个组件把所有你的 CSS 加载到入口主文件中来做任何事情。

1、加载器

- css-loader：遍历 CSS 文件，然后找到 url() 表达式然后处理。
- style-loader：把原来的 CSS 代码插入页面中的一个 style 标签中。

下载安装加载器：

	npm install css-loader --save-dev
	npm install style-loader --save-dev

2、webpack.config.js配置

	{
      test: /\.css$/, // Only .css files
      loader: 'style!css' // Run both loaders
    }

3、加载css方式

- 合并加载，放在入口js下引入。
- 懒加载，每个组件引入自身定制的css文件，使用不同命名空间。
- 内联样式style

>注！当 Webpack-dev-server 在 浏览器自动刷新 下运行的时候，CSS 也会自动更新，不过有点不同的是，当你改变了一个 CSS 文件，属于那个文件的标签会更新新的内容但不会刷新。

### 加载LESS和SASS ###

安装less或sass模块

	npm install less --save
	npm install sass --save

1、加载器

less-loader 或 sass-loader

	npm install less-loader --save-dev
	或
	npm install sass-loader --save-dev

2、webpack.config.js配置

	 // LESS
    {
      test: /\.less$/,
      loader: 'style!css!less'
    },

    // SASS
    {
      test: /\.scss$/,
      loader: 'style!css!sass'
    }

3、引入

- 相对或绝对路径，类似 css 引入 ，webpack会自动找出，并识别依赖。
- 也可以直接从 node_module 中加载less。
		
		import '~bootstrap/less/bootstrap'


### 内联Image ###

直到 HTTP/2 你才能在应用加载的时候避免设置太多 HTTP 请求。根据浏览器不同你必须设置你的并行请求数，如果你在你的 CSS 中加载了太多图片的话，可以自动把这些图片转成 BASE64 字符串然后内联到 CSS 里来降低必要的请求数，这个方法取决与你的图片大小。你需要为你的应用平衡下载的大小和下载的数量，不过 Webpack 可以让这个平衡十分轻松适应。

	
	
1、安装
会将路径转为BASE64字符串；把 CSS 中的 “url()” 像其他 require 或者 import 来处理。

安装url-loader

	npm install url-loader --save-dev

安装file-loader 

	npm install file-loader --save-dev


2、配置

	{
      test: /\.(png|jpg)$/,
      loader: 'url?limit=25000'
    }

url-loader 传入的 limit 参数是告诉它图片如果不大于 25KB 的话要自动在它从属的 css 文件中转成 BASE64 字符串。

### 内联字体 ###

就像内联图片一样来内联字体。

字体类别：(eot | woff | woff2 | ttf | svg | png | jpg)等

配置 

	{
      test: /\.(eot|woff|woff2|ttf|svg|png|jpg)(\?v=[\d\.]+)?$/,
      loader: 'file?name=files/[hash].[ext]'
    }
	或
	 {
      test: /\.woff$/,
      loader: 'url?limit=100000'
    }

## 部署策略 ##

### 发布配置 ###

1、配置package.json脚本，设置npm run deploy

	"scripts": {
	    "deploy": "NODE_ENV=production webpack -p --config webpack.production.config.js"
	  },

用生产(线上)参数运行 Webpack 来指向另一个配置文件。我们也使用了环境变量 “production” 来让我们的模块自动去优化。

2、webpack.config.js配置案例

	var path = require('path');
	var node_modules_dir = path.resolve(__dirname, 'node_modules');
	
	var config = {
	  entry: path.resolve(__dirname, 'app/main.js'),
	  output: {
	    path: path.resolve(__dirname, 'dist'),
	    filename: 'bundle.js'
	  },
	  module: {
	    loaders: [{
	      test: /\.js$/,
	
	      // There is not need to run the loader through
	      // vendors
	      // 这里再也不需通过任何第三方来加载
	      exclude: [node_modules_dir],
	      loader: 'babel'
	    }]
	  }
	};
	
	module.exports = config;

3、运行

	npm run deploy 

webpack运行生成模式

### 合并成单文件 ###

单入口模式：

- 应用很小
- 更新应用很少
- 不关心初始加载时间

webpack.production.config.js

	var path = require('path');
	var config = {
	  entry: path.resolve(__dirname, 'app/main.js'),
	  output: {
	    path: path.resolve(__dirname, 'dist'),
	    filename: 'bundle.js'
	  },
	  module: {
	    loaders: [{
	      test: /\.js$/,
	      loader: 'babel'
	    }]
	  }
	};
	
	module.exports = config;

### 分离应用和第三方依赖 ###

1、条件

- 当你的第三方的体积达到整个应用的 20% 或者更高的时候。
- 更新应用的时候只会更新很小的一部分
- 你没有那么关注初始加载时间，不过关注优化那些回访用户在你更新应用之后的体验。
- 有手机用户。

2、webpack.production.config.js

	var path = require('path');
	var webpack = require('webpack');
	var node_modules_dir = path.resolve(__dirname, 'node_modules');
	
	var config = {
	  entry: {
	    app: path.resolve(__dirname, 'app/main.js'),
	
	    // Since react is installed as a node module, node_modules/react,
	    // we can point to it directly, just like require('react');
	    // 当 React 作为一个 node 模块安装的时候，
	    // 我们可以直接指向它，就比如 require('react')
	    vendors: ['react']
	  },
	  output: {
	    path: path.resolve(__dirname, 'dist'),
	    filename: 'app.js'
	  },
	  module: {
	    loaders: [{
	      test: /\.js$/,
	      exclude: [node_modules_dir],
	      loader: 'babel'
	    }]
	  },
	  plugins: [
	    new webpack.optimize.CommonsChunkPlugin('vendors', 'vendors.js')
	  ]
	};
	
	module.exports = config;

这些配置会在 dist/ 文件夹下创建两个文件：app.js 和 vendors.js，两个文件均需要引入到html中。


### 多重入口 ###

1、应用条件

- 你的应用有多种不同的用户体验，但是他们共享了很多代码。
- 你有一个使用更少组件的手机版本
- 你的应用是典型的权限控制，你不想为普通用户加载所有管理用户的代码。

2、webpack.production.config.js

	var path = require('path');
	var webpack = require('webpack');
	var node_modules_dir = path.resolve(__dirname, 'node_modules');
	
	var config = {
	  entry: {
	    app: path.resolve(__dirname, 'app/main.js'),
	    mobile: path.resolve(__dirname, 'app/mobile.js'),
	    vendors: ['react'] // 其他库
	  },
	  output: {
	    path: path.resolve(__dirname, 'dist'),
	    filename: '[name].js' // 注意我们使用了变量
	  },
	  module: {
	    loaders: [{
	      test: /\.js$/,
	      exclude: [node_modules_dir],
	      loader: 'babel'
	    }]
	  },
	  plugins: [
	    new webpack.optimize.CommonsChunkPlugin('vendors', 'vendors.js')
	  ]
	};
	
	module.exports = config;

这个配置会在 dist/ 文件夹下创建三个文件：app.js、mobile.js和vendors.js，大部分的代码在mobile.js文件中，也有一部分在 app.js 中，不过这是我们需要的，我们不会在同一个页面中同时加载 app.js 和 mobile.js。


### 懒加载入口文件 ###

1、运用条件

- 你有一个相对比较大的应用，可以让用户可以访问应用的不同部分。
- 你非常关注初始渲染时间

2、结合 react-router 或 es6 require.ensure，通过路由实现懒加载。

	import React from 'react';
	import Feed from './Feed.js';
	
	class App extends React.Component {
	  constructor() {
	    this.state = { currentComponent: Feed };
	  }
	  openProfile() {
	    require.ensure([路由], () => {
	      var Profile = require('./Profile.js');
	      this.setState({
	        currentComponent: Profile
	      });
	    });
	  }
	  render() {
	   return (
	      return <div>{this.state.currentComponent()}</div>
	    );
	  }
	}
	React.render(<App/>, document.body);

>不需要在config配置中设置懒加载依赖，Webpack 会自动理解他们，然后分析你的代码


## 插件篇 ##

### 自动补全CSS3前缀——autoprefixer ###

>官方是这样说的：Parse CSS and add vendor prefixes to CSS rules using values from the Can I Use website
，也就是说它是一个自动检测兼容性给各个浏览器加个内核前缀的插件。

**举个栗子**：最新的弹性盒模型flux
实际代码：

	:fullscreen a {
	    display: flex
	}
插件自动补充后

	a {
	    display: -webkit-box;
	    display: -webkit-flex;
	    display: -ms-flexbox;
	    display: flex
	}
效果显而易见，我们可以更专注于css布局和美化，而不需要花过多的精力都写相同的外码而加上不同的前缀，也减少了冗余代码。

1、安装下载

	npm install autoprefixer --save-dev
	npm install postcss-loader --save-dev

2、配置

	var autoprefixer = require('autoprefixer');
	module.exports={
	  //其他配置这里就不写了
	
	  module:{
	    loaders:[
	    {
	      test:/\.css$/,
	      //在原有基础上加上一个postcss的loader就可以了
	      loaders:['style-loader','css-loader','postcss-loader']
			/*{
	      test: /\.css$/, // Only .css files
	      	loader: 'style!css!postcss' // loaders
	      }*/
	    }]
	  },
	  postcss:[autoprefixer({browsers:['last 2 versions']})]
	
	}


![个人微信公众号](http://upload-images.jianshu.io/upload_images/2455352-daf8ead7e17f1161.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)