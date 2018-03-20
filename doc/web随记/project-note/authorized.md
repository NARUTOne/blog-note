# 系统权限
> 本文主要是总结前段时间项目中权限管理这块开发设计，包括：资源（路由级）权限、操作（按钮级）权限，以及登录后用户级别的权限分配。当然还有数据级权限，这个暂时没有加入，最后会简单提出个解决思路。

## 背景

公司需要做一款产品，里面需要有一个平台用来类似手机APP似的房子不同的子产品入口（类快捷图标），各子产品间实现单点登录，创建不同账户级别，可以分配产品权限，产品资源权限，产品操作级权限。

![](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/project-note/authorized/app.png)

本产品，最后权限做了双重控制，前后端都控制， 本文只从前端角度进行总结。

- 单点登录采用CAS
- 前端技术采用React技术栈：使用了一个自建的简易脚手架[FireLeaf-React-Scaffold](https://github.com/NARUTOne/FireLeaf-React-Scaffold)

## 登录（产品）权限
> 此产品因为涉及到数据库表，用户管理，数据调度等一系列子产品功能，因此设计上用户分为不同级别，拥有不同产品权限，不同的账户登录，也就展示的不同。

账户所拥有的产品权限信息，登录后后台将会返回数组形式，每项包含一些信息，至于这些产品信息管理，也在后台系统中进行统一管理配置，之后将会在资源权限提及。

```json
[{
  id: "1",
  isLeaf:0,
  name:"用户管理中心",
  nodeId:"node0",
  pOrgCode:"",
  uri:"",
  url:"/uc"
}]
```
其中，主要是url来进行跳转，这里有个问题：url里的路径有时是同一域名下的产品，也可以是一些以前的产品路径，这就需要进行url判断

```js
handleALink = (href) => {
  const regex = /(https?:\/\/)?(\w+\.?)+(\/[a-zA-Z0-9\\?%=_\-\\+\\/]+)?/gi;
  const url = href.replace(regex, function (match, capture) {  
    if (capture) {  
      return match;
    }  
    else {  
      return 'http://' + match;  
    }  
  });

  const a = document.createElement('a');
  a.href = url;
  a.target = '_blank';
  a.click();
  window.URL.revokeObjectURL(url);
}
```
当然，有人会问，如果直接进入一个产品地址，如何判断登录呢？
我们前后端约定好一个未登录code码 `401`

```js
...
xhr.success = (res, options) => {
  if (typeof res !== 'object') {
    message.error( apiUrl + ': response data should be JSON');
    return;
  }
  switch (res.code) {
    case 200:
      options.success && options.success(res);
      break;
    case 401:
      auth.destroy();
      message.error('请登录！');
      toLogin(res.url);
      break;
    default:
      message.error(res.message || 'unknown error');
  }
};
...

import auth from 'src/utils/auth';

function toLogin(url) {
  auth.destroy();
  localStorage.clear();
  
  const toHref = url;
  window.location.href = toHref;
}

export default toLogin;
```
其实前端也将登录后的用户信息存入了localstorage, 退出登录后将会销毁，这也能进行登录验证，但是不是很准确；当然这里其实还得进行路由权限验证，这下面将会讲到。

## 资源（路由）权限
> 登录后，进入某产品后,将会获得此产品所拥有的资源权限树形数据，将用来渲染一些导航栏，或者跳转导航。

路由权限设计有些考虑的问题：

- 路由主要分为这几类：未登录可访问、登录显示导航中（显性）、登录未显示在导航中（隐性）
- 未登录后需要跳转到未登录访问首页，当然此产品没有，直接跳到单点登录页
- 已登录后，跳转无权限路由，将转到404页

**后台系统资源管理设计**

![](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/project-note/authorized/resource.png)
![](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/project-note/authorized/resource-info.png)

资源管理采用树形结构，同级叶子可以进行拖拽调换位置展示导航菜单，每级叶子均可以添加叶子，删除修改。叶子的信息这里有些特有的设计：

- 节点类别： 
  - 外链项目：产品项目属于需要外来的，挂载在平台下，webUrl将要填写完整地址
  - 项目根：产品项目
  - 资源根：导航级的根
  - 资源叶子：导航级叶子（显性），如果此类型下还有叶子，那将属于隐性路由
- web类别：菜单级（显性），内容级（隐性）
- 资源操作类型：每个路由级界面下的操作权限配置

> **对于此颗资源树的数据操作保存的管理开发，其实当产品越来越多，树将会越来越繁杂，层级节点将会越来越多，需要将树进行产品级的保存及展示优化**

登录后，对显示菜单进行渲染后，要对访问的路由进行访问权限审核检查：

```js
// react-router
<Router
    history={browserHistory}
  >
    <Route path={PName}
      onEnter={(...args) => {
      requireAuth(...args);
    }}
    component={App} 
    breadcrumbName="/">
     ....
    </Route>
</Router>

// 用户登录验证, 路由权限检查
function requireAuth(nextState, replace) {
  const path = nextState.location.pathname;

  if(auth.isLoginIn()) {
    if(!checkRouter(path)) {
      replace({
        pathname: PName + '/404',
        state: {
          referrer: path
        }
      });
    }
  }  
}

// 入口app.js中检测

componentWillReceiveProps(nextProps) {
  const { location } = nextProps;
  // URL 发生改变的时候检查是否有权限访问
  if (location.pathname === '/' || location.pathname !== this.props.location.pathname) {
    checkRouter(routers);
  }
}

```

```js
/**
 * 路由检测
 * @param {url_auth} obj
 * @param {path} str 
 */
import {PName} from 'utils/config';
import auth from 'utils/auth';

function isNumber(obj) {
  var num = obj - 0;
  return typeof num === 'number' && !isNaN(num);  
}
 
const checkRouter = (path) => {
  const resource = auth.getResource();
  const {urlAuth} = resource;
  const urlList = [].concat(urlAuth);
  urlList.push(PName + '/404');
  
  const arr = urlList.filter(item => {
    const arr = path.split('/');
    if(isNumber(arr[arr.length - 1])) {
      arr.pop();

      return item.indexOf(arr.join('/')) > -1;
    }
    else {
      return path == item;
    }
  });

  return !!arr.length;
};

export default checkRouter;
```

## 操作（按钮级）权限
> 每个路由界面级均有操作权限配置信息，因此需要对所有页面的按钮级操作进行权限配置，主要通过显隐操作来控制

**操作权限管理界面**

![](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/project-note/authorized/operateType.png)

操作权限主要是设计了一个webkey配置，方便前端的操作权限的检测。操作权限是进行统一管理的，路径资源管理下可以进行操作权限的勾选配置。
操作权限由于涉及到按钮级，也就是组件级，不能在每个页面单独配置，那样需求改动，将会陷入深坑。我采用的[HOC](https://reactjs.org/docs/higher-order-components.html)高阶组件的封装套路：

```js
/**
 * 权限 高阶组件
 * @param {reactNode} ComposedComponent 需要权限控制的组件
 * @param {string} path 页面pathname
 * @param {string} webKey 操作web key
 */

import React, {Component} from 'react';
import PropTypes from 'prop-types';
import auth from 'utils/auth';
import tools from 'utils/tools';

const {getNodeByKeys} = tools;

const wrapAuth = (ComposedComponent, path) => class WrapComponent extends Component {
  // 构造
  constructor(props) {
    super(props);
    this.state = {
      webKey: props.webKey
    };
  }

  checkBtnAuth(webKey) {
    const {navList} = auth.getResource();
    const keys = [];
    keys.push(path);
    let bol = true;

    const nodes = getNodeByKeys(navList, keys, 'url');

    if(nodes.length) {
      const indexNode = nodes[0];
      const types = indexNode.type;

      if(types) {
        const type = types.find(item => item.webKey == webKey);
        if(!type) {
          bol = false;
        }
      }
    }

    return bol;
  }

  static propTypes = {
    webKey: PropTypes.string.isRequired, // 按钮级的权限webKey
  };

  render() {
    const {webKey, ...others} = this.props;
    if (this.checkBtnAuth(webKey)) {
      return <ComposedComponent  { ...others} />;
    } else {
      return null;
    }
  }
};

export default wrapAuth;

```
界面中使用也是很简单：

```js
// 操作权限
const authPath = PName + '/pm/pmTable';
const AuthButton = wrapAuth(Button, authPath);
...
<AuthButton webKey={AUTH_BTN_KEY.ADD} icon='plus' type="primary" onClick={this.handleAdd}>新建</AuthButton>

```

这样采用HOC进行封装，可以进行一些别的需要扩展：加入操作动画，改变样式等。

## 数据（接口）权限

不同的用户登录以后，对数据范围的权限是有限制的，那些能够访问，那些不能访问在产品设计的是就已经定义好，当访问一个当前登录用户无权访问的 API 或者数据的时候，API 响应中会返回对应的 code, 这个 code 是提前就前后的约定好的值。
**这部分权限需要在`xhr api`层调用接口时进行数据权限的判断**

## 总结

总结一下，其实前端在做权限控制的时候，依赖于后端 API 返回的配置信息，所以在权限设计，路由设计，数据结构设计的时候，前后端一定要约定好。
