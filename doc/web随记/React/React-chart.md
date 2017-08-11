# React 可视化

> 在学习使用过React一段时间后，感觉React的使用还是比较舒适的。了解React的组件生命周期，路由，及通信你会发现React对于开发真的很便利。

**引言**
偶然间，看见一篇博客关于[React可视化的](https://me-momo.github.io/kasmine.blog/2017/07/03/%E7%8E%A9%E8%BD%ACreact%E5%8F%AF%E8%A7%86%E5%8C%96.html)，自己本身这半年多使用React开发也有一些感触，就借鉴这篇博客写下这篇随文。

> 参考： [React可视化的](https://me-momo.github.io/kasmine.blog/2017/07/03/%E7%8E%A9%E8%BD%ACreact%E5%8F%AF%E8%A7%86%E5%8C%96.html)
## 推荐

**脚手架**
- dva-cli： https://github.com/dvajs/dva/blob/master/README_zh-CN.md
- create-bfd-app： https://github.com/baifendian/create-bfd-app
- create-react-app: https://github.com/facebookincubator/create-react-app

**React组件库**
- Antd Design： https://ant.design/index-cn
- bfd-ui： http://ui.baifendian.com/bian

## React 可视化

> 回到正题。这篇主要是介绍些React中的可视化使用。

说到可视化，无外乎使用原生的canvas、svg或一些可视化库进行可视化开发。

![](http://i.imgur.com/CoPY36J.jpg)

### canvas

```js
class Graphic extends React.Component {
  constructor(){
    super();
    this.state = {
      color: 'green'
    }
    //this.onChange = this.onChange.bind(this);
  }

  componentDidMount(){
    const context = ReactDOM.findDOMNode(document.getElementById('canvas')).getContext('2d');
    this.paint(context);
  }
componentDidUpdate(){
    const context = ReactDOM.findDOMNode(document.getElementById('canvas')).getContext('2d');
  context.clearRect(0, 0, 200, 200);
    this.paint(context);
  }
  paint(context) {
    context.save();
    context.fillStyle = this.state.color;
    context.fillRect(0, 0, 100, 100);
    context.restore();
  }
  onChange(e){
    this.setState({
      color: e.target.value
    });
  }
  render(){
    return (
      <div>
        <canvas id='canvas' width='200' height='200' style= ></canvas>
        <input type='color' value={this.state.color} onChange={::this.onChange}/>
       </div>
    )
  }
}
ReactDOM.render(
  <Graphic />,
  document.getElementById('root')
);
```

### svg

```js
class SVG extends React.Component {
  constructor(){
    super();
  
  }
  render(){
    return (
      <svg width='200' height='200' viewBox='0 0 200 200'>
           <circle cx='50' cy='50' r='20'/>
           <circle cx='100' cy='50' r='20'/>
           <circle cx='75' cy='100' r='40'/>
           <circle cx='60' cy='90' r='10' fill='#fff' />
           <circle cx='90' cy='90' r='10' fill='#fff' />
      </svg>
    )
  }
}
ReactDOM.render(
  <SVG />,
  document.getElementById('root')
);
```

## 可视化组件包装
> 利用一些可视化库插件进行包装

### Echarts 

- echarts配置文件
- echarts渲染

**options**
```js
const LineScore = (data) => {
	const option = {
    title: {
        text: '近年消费趋势'
    },
    tooltip: {
        trigger: 'axis'
    },
    legend: {
        data:['clothes','foods','home','travel']
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    toolbox: {
        feature: {
            saveAsImage: {}
        }
    },
    xAxis: {
        type: 'category',
        boundaryGap: false,
        data: ['2010','2011','2012','2013','2014','2015','2016']
    },
    yAxis: {
        type: 'value',
        name:'￥（元）'
    },
    series: [
        {
            name:'clothes',
            type:'line',
            smooth: true,
            data:[1200, 1320, 1010, 1340, 1900, 2300, 2100]
        },
        {
            name:'foods',
            type:'line',
            smooth: true,
            data:[2200, 1820, 1910, 2340, 2900, 3300, 3200]
        },
        {
            name:'home',
            type:'line',
            smooth: true,
            data:[800, 1020, 1210, 1540, 1800, 2300, 3100]
        },
        {
            name:'travel',
            type:'line',
            smooth: true,
            data:[600, 920, 1010, 1340, 1700, 2300, 2230]
        }
    ]
	};
 
	return option
}
export {
	LineScore
}
```
**使用**

```js
import {LineScore} from './chartsOption'
import {EchartsChart} from 'components'
	
const ChartDom = (props) => {
	const chart_options = LineScore()
	return(
		<div style={{height:'300px'}}>
			<EchartsChart options={chart_options}/>
		</div>
	)
}

ReactDOM.render(
  <ChartDom />,
  document.getElementById('root')
);

```

**组件开发**

```bash
EchartsChart
|- index.js   组件的入口文件
|- index.less  组件的样式
|- main.js  组件的可视化库包装
```

- index.js , 里面使用的一些判断非空 及组件渲染优化，想了解都可以fork我的[antd-demo](https://github.com/wz-one-piece/Antd-Demo)

```js
import React, {Component} from 'react'
import PropTypes from 'prop-types'
import ReactDM from 'react-dom'
import classnames from 'classnames'
import Chart from './main'
import styles from './index.less'
import {tools, shouldComponentUpdate} from 'utils'

class EchatsChart extends Component {
	constructor(props) {
		super();
		this.state = {}	
	}

	componentDidMount() {
		this.container = ReactDM.findDOMNode(this.refs.echarts)
		if(this.props.options && !tools.emptyObj(this.props.options)) {

			this.renderChart(this.props)
		}
	}

	componentWillReceiveProps(nextProps) {
		if('options' in nextProps && nextProps.options != this.props.options) {
			
			this.renderChart(nextProps)
		}
	}

	shouldComponentUpdate = shouldComponentUpdate

	renderChart(props) {
		new Chart({
			container:this.container,
			...props
		})
	}

	render() {
		return (
			<div className={styles.echarts_chart}>
				<div className={styles.echarts_chart_box} ref='echarts'>
				</div>
			</div>
		)
	}
}

export default EchatsChart;
```
- main.js ,组件的核心，可视化实现

```js
/*
  chart base on echarts
*/
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import echarts from 'echarts'
import {tools, request} from 'utils'

class Chart extends Component {
  constructor(config) {
    super()
    this.config = config;
    this.container = config.container;
    
    this.init()
  }
  
  init() {
  	const _this = this
  	if(_this.config.showMapName) { //判断是否需要展示地图
    	
    	async function getMap(){
			  return request({
			    url: `/data/map/${_this.config.showMapName}.json`,
			    method: 'get',
			    data: {},
			  })
			}
    	
    	getMap().then(res => {
    		console.log(res)    		
				res && echarts.registerMap(_this.config.showMapName, res);
				_this.renderChart()
			});		
			
    }
  	else {
  		_this.renderChart()
  	}
  }
  
  renderChart() {
  	const myChart = echarts.init(this.container);

    if(this.config.options && !tools.emptyObj(this.config.options)) {
      this.config.options.color = ['#64ea91','#8fc9fb', '#d897eb', '#f69899', '#f8c82e','#f797d6',  '#ca8622', '#bda29a','#6e7074', '#546570', '#c4ccd3']
      myChart.setOption(this.config.options);
    }

    window.addEventListener("resize",function(){ //响应式
      myChart.resize();
    });

    const _this = this
    for (let key in this.config) { //添加事件监听
      if(/^on[a-zA-Z]*$/.test(key)) {
        const even = key.substring(2);
        myChart.on(even, function (params) {
          _this.config[key] && _this.config[key](params)
        });
      }
    } 
  }
}
Chart.PropTypes = {
  container: PropTypes.object,
  showMapName:PropTypes.string,
  options: PropTypes.object,
  customProp(props) {
    if(!props.options) {
      return new Error('You echarts chart need a options!')
    }
  }
}

export default Chart

```

>我以前编写echarts类型得组件，是将一种类型单独编写组件的，但后来嫌经常需要做些重复的编码工作，所以就合并了。如果你需要单独编写，当然这样个性化，独立性更好，维护好。可以查看https://zhuanlan.zhihu.com/p/28331793

## d3JS

> 当然现在d3已经到v4了，支持canvas了。我这实现和[React可视化的](https://me-momo.github.io/kasmine.blog/2017/07/03/%E7%8E%A9%E8%BD%ACreact%E5%8F%AF%E8%A7%86%E5%8C%96.html)不同。

- D3支持数据和节点绑定，数据变化，相应的节点也发生变化。React推崇单向数据流，数据从父组件到子组件，每个子组件只实现简单的一个模块
- D3实现一套selector机制， 能够让开发址直接操作DOM节点、SVG节点。React使用虚拟DOM和高性能DOM diff算法，开发者无须关注节点操作。

```js
import d3 from 'd3'

/*
	base on  d3.js-v3
*/

class AtlasChart {
	constructor(props) {
		this.props = props;
		this.container = props.container;
		this.orgData = props.atlasData;
		this.width = (props.options && props.options.width) || this.container.clientWidth;
		this.height = (props.options && props.options.height) || this.container.clientHeight;

	  const defaultOption = {
	    zoom: {
	      scale:0.8, 
	      x:this.width/2, 
	      y:this.height/2
	    }
	  }
		this.options = Object.assign(defaultOption, props.options);

		this.init();
	}

	init() {
		const chartData = this.orgData;
		this.renderChart(chartData);
	}

	renderChart(chartData) {
		const _this = this;
		const W = this.width,
			H = this.height,
			rx = W / 2,
			ry = H / 2; 
		const n = {
      margin: 0,
      radiusR: 130
    };
    const Roate = _this.options.zoom.rotate;

		const color = ['#1f77b4', '#778ae6', '#b46bc5', '#eda61d', '#c3d41b', '#91dc8a', '#24a6da', '#aec7e8', '#ff9896', '#2ca02c'];

		d3.select(this.container).html('');

		var cluster = d3.layout.cluster()
    .size([360, ry - 30])
    .separation(function(e, t) {
      return (e.parent == t.parent ? 1 : 2) / e.depth
    })
    .sort(null);

    var lineScale = d3.scale.linear().domain([0, 2]).range([10, 7]);
		var diagonal = d3.svg.diagonal.radial()
    	.projection(function(d) { return [d.y, d.x / 180 * Math.PI]; });

		function zoom() {
			// console.log(d3.event)
      _this.options.zoom.x = d3.event.translate[0];
      _this.options.zoom.y = d3.event.translate[1];
      _this.options.zoom.scale = d3.event.scale;
    
      svg_center.attr("transform", "translate(" + d3.event.translate + ")scale(" + d3.event.scale + ")");
    }

		const zoomListener = d3.behavior.zoom()
      .scaleExtent([0.1, 3])
      .scale(_this.options.zoom.scale)
      .translate([_this.options.zoom.x,_this.options.zoom.y])
      .on("zoom", zoom);

    // 自定义提示框
    let tip = d3.select(this.container)
      .append("div")
      .attr('class','tooltips');
		
		let svg = d3.select(this.container)
			.append('svg')
			.attr('width', W)
			.attr('height', H)
			.attr('id', 'atlas-chart')
			.call(zoomListener)
			.on('dblclick.zoom', null);

		let svg_center = svg.append('g')
			.attr('id','svg_center')
			.attr("transform", "translate(" + [_this.options.zoom.x, _this.options.zoom.y] + ")scale(" + _this.options.zoom.scale + ")" );

		let nodes = cluster.nodes(chartData);

		nodes.forEach(function(d) {
      d.y = n.radiusR * d.depth,
      d.depth != 0 && (d.x += Roate,
      d.x >= 360 ? d.x -= 360 : d.x < 0 && (d.x += 360))
    });

		let link = svg_center.selectAll("path.link")
      .data(cluster.links(nodes))
      .enter().append("svg:path")
      .attr("class", "link")
      .attr("d", diagonal)
      .style('stroke', d => {
      	return d.target.color ? d.target.color : color[d.target.group]
      });

  	let node = svg_center.selectAll("g.node")
      .data(nodes)
      .enter().append("svg:g")
      .attr("class", "node")
      .attr("transform", function(d) { return d.parent ? "rotate(" + (d.x - 90) + ")translate(" + d.y + ")" : ''; })
      .on('mouseover', (d) => {
        d3.event.stopPropagation();
        tip.html(`${d.node_name}`)
          .style('left', `${d3.mouse(this.container)[0] + 20}px`)
          .style('top', `${d3.mouse(this.container)[1] + 20}px`)
          .style('display', 'block')
      })
      .on('mousemove', () => {
        tip.style('left', `${d3.mouse(this.container)[0] + 20}px`)
          .style('top', `${d3.mouse(this.container)[1] + 20}px`)
      })
      .on('mouseout', () => {
        tip.style('display', 'none')
      });

    node.append("svg:circle")
      .attr("r", 6)
      .attr('class', d => {
      	return d.depth == 1 || d.depth == 0 ? 'node_breath': ''
      })
      .style('fill', d => {
      	return d.color? d.color : color[d.group]
      });

  	node.append("svg:text")
  		.attr('class', 'node-text')
      .attr("dx", function(d) { return d.x < 180 ? 8 : -8; })
      .attr("dy", d => {
      	return d.depth?".31em":'-1.5em'
      })
      .attr("text-anchor", function(d) { return d.parent ? d.x < 180 ? "start" : "end" : "start"; })
      .attr("transform", function(d) { 
      	let t =  "rotate(180)"
      	if(d.depth) {
      		t = d.x < 180 ? null : "rotate(180)"; 
      	}
      	else {
      		t = 'translate(' + (-(d.node_name.length * 12 / 2)) + ")";
      	}

      	return t
      })
      .text(function(d) { 
      	return d.node_name.length > 10 ? d.node_name.slice(0, 10) + '...' : d.node_name      	
      });

	}
}

export default AtlasChart
```
以上代码实现的是一种关系图谱。
![](http://i.imgur.com/YHYjS3V.png)

## Recharts
>这是一种将各种图表进行细分为标签组件进行组合形成可视化。
>http://recharts.org/#/en-US/api

**Recharts优点**
- 声明式标签
- 贴近原生SVG配置
- 接口式API，解决各种个性化需求

```js
const {LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend} = Recharts;
const data = [
      {name: 'Page A', uv: 4000, pv: 2400, amt: 2400},
      {name: 'Page B', uv: 3000, pv: 1398, amt: 2210},
      {name: 'Page C', uv: 2000, pv: 9800, amt: 2290},
      {name: 'Page D', uv: 2780, pv: 3908, amt: 2000},
      {name: 'Page E', uv: 1890, pv: 4800, amt: 2181},
      {name: 'Page F', uv: 2390, pv: 3800, amt: 2500},
      {name: 'Page G', uv: 3490, pv: 4300, amt: 2100},
];
const SimpleLineChart = React.createClass({
	render () {
  	return (
    	<LineChart width={600} height={300} data={data}
            margin={{top: 5, right: 30, left: 20, bottom: 5}}>
       <XAxis dataKey="name"/>
       <YAxis/>
       <CartesianGrid strokeDasharray="3 3"/>
       <Tooltip/>
       <Legend />
       <Line type="monotone" dataKey="pv" stroke="#8884d8" activeDot={{r: 8}}/>
       <Line type="monotone" dataKey="uv" stroke="#82ca9d" />
      </LineChart>
    );
  }
})

ReactDOM.render(
  <SimpleLineChart />,
  document.getElementById('container')
);
```
>如果想了解下Recharts的几大特性，可以看看https://zhuanlan.zhihu.com/p/20641029


## 题外

组件化这些思想不只是在可视化组件中可以应用，很多组件也可以考虑利用这种思想来实现，例如表格组件就可以抽取Table 和 Column 两个组件，然后大家使用表格也非常简单：

```js
<Table data={data}>
  <Column name="名称" dataKey="name"/>
  <Column name="数量" dataKey="count" align="right" th={<SortableTh order="asc" onChange={handleSort}/>}/>
  <Column name="金额" dataKey="amt" td="float" align="right"/>
</Table>
```

## 最后

放出我的学习antd时做的一个小demo,里面有这上面的可视化组件代码，当然还有一些其他antd的开发套路。

[antd-demo](https://github.com/wz-one-piece/Antd-Demo)

**欢迎指导O(∩_∩)O哈哈~**