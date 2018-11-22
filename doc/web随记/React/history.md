# react-router v4 使用 history 控制路由跳转

[参考](https://github.com/brickspert/blog/issues/3)

## 背景

当我们使用`react-router v3`的时候，我们想跳转路由，我们一般这样处理

我们从`react-router`导出`browserHistory`。
我们使用`browserHistory.push()`等等方法操作路由跳转。
类似下面这样

```js
import browserHistory from 'react-router';

export function addProduct(props) {
  return dispatch =>
    axios.post(`xxx`, props, config)
      .then(response => {
        browserHistory.push('/cart');  // 这里
      });
}
```
but!! 🐛 问题来了，在`react-router v4`中，不提供`browserHistory`等的导出~~

## 解决方案

### 使用 withRouter

`withRouter`高阶组件，提供了history让你使用~

```js
import React from "react";
import {withRouter} from "react-router-dom";

class MyComponent extends React.Component {
  ...
  myFunction() {
    this.props.history.push("/some/Path");
  }
  ...
}
export default withRouter(MyComponent);
```

这是官方推荐做法哦。但是这种方法用起来有点难受，比如我们想在`redux`里面使用路由的时候，我们只能在组件把`history`传递过去。

就像问题章节的代码那种场景使用，我们就必须从组件中传一个`history`参数过去。。。

###  使用 Context
`react-router v4 `在 `Router` 组件中通过`Contex`暴露了一个`router`对象~

在子组件中使用`Context`，我们可以获得`router`对象，如下面例子~

```js
import React from "react";
import PropTypes from "prop-types";

class MyComponent extends React.Component {
  static contextTypes = {
    router: PropTypes.object
  }
  constructor(props, context) {
     super(props, context);
  }
  ...
  myFunction() {
    this.context.router.history.push("/some/Path");
  }
  ...
}
```
当然，这种方法慎用~尽量不用。因为`react`不推荐使用`contex`哦。在未来版本中有可能被抛弃哦。

###  hack
其实分析问题所在，就是v3中把我们传递给`Router`组件的`history`又暴露出来，让我们调用了~~

而`react-router v4 `的组件`BrowserRouter`自己创建了`history`，
并且不暴露出来，不让我们引用了。尴尬~

我们可以不使用推荐的`BrowserRouter`，依旧使用`Router`组件。我们自己创建`history`，其他地方调用自己创建的`history`。看代码~

我们自己创建一个`history`

```js
// src/history.js

import createHistory from 'history/createBrowserHistory';

export default createHistory();
```

我们使用`Router`组件

```js
// src/index.js

import { Router, Link, Route } from 'react-router-dom';
import history from './history';

ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      ...
    </Router>
  </Provider>,
  document.getElementById('root'),
);
```

其他地方我们就可以这样用了
```js
import history from './history';

export function addProduct(props) {
  return dispatch =>
    axios.post(`xxx`, props, config)
      .then(response => {
        history.push('/cart'); //这里
      });
}
```

### 非要用BrowserRouter
确实，`react-router v4`推荐使用`BrowserRouter`组件，而在第三个解决方案中，我们抛弃了这个组件，又回退使用了`Router`组件。

怎么办。 你去看看[BrowserRouter](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/modules/BrowserRouter.js)的源码，我觉得你就豁然开朗了。

源码非常简单，没什么东西。我们完全自己写一个`BrowserRouter`组件，然后替换第三种解决方法中的`Router`组件。嘿嘿 😃 。