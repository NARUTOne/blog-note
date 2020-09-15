# 高阶组件 HOC

> 高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件。

## 高阶函数模拟

高阶函数就是一个接收一个函数并返回另外一个新的函数！

```js
function welcome(username) {
    console.log('welcome ' + username);
}

function goodbey(username) {
    console.log('goodbey ' + username);
}

function wrapWithUsername(wrappedFunc) {
    let newFunc = () => {
        let username = localStorage.getItem('username');
        wrappedFunc(username);
    };
    return newFunc;
}

welcome = wrapWithUsername(welcome);
goodbey = wrapWithUsername(goodbey);

welcome();
goodbey();
```

wrapWithUsername函数就是一个“高阶函数”, 处理了username，传递给目标函数

## 高阶组件——代理

没有副作用的纯函数

- 操作`props`
- 访问`ref`（不推荐）
- 抽取状态
- 包装组件

### props

**删除props**:

```js
import React from 'react';

function HOCRemoveProps (WrappedComponent) {
  return class WrappingComPonent extends React.Component {
    render () {
      const {user, ...otherProps} = this.props;
      // 去除 user
      return <WrappedComponent {...otherProps} />
    }
  }
}
export default HOCRemoveProps;
```

**增加props**:

```js
import React from 'react';

function HOCAddProps (WrappedComponent, uid) {
  return class WrappingComPonent extends React.Component {
    render () {
      const newProps = {
        uid
      };
      return <WrappedComponent {...this.props} {...newProps} />
    }
  }
}
export default HOCAddProps;
```

### 抽取状态

将所有的状态的管理交给外面的容器组件，这个模式就是 抽取状态

```js
import React from 'react';

const HOCContainer = (WrappedComponent) => {
  class extends React.Component {
    constructor (props) {
      super(props);
      this.state = {
        name: ''
      }
    }

    onNameChange = (event) => {
      this.setState({
        name: event.target.value
      });
    }

    render () {
      const newProps = {
        value: this.state.name,
        onChange: this.onNameChange
      };

      return <WrappedComponent {...this.props} {...newProps} />
    }
  }
}

export default HOCContainer;

```

***

```js
@HOCContainer

class SampleComponent extends React.Component {
  render () {
    return <input name="name" {this.props.name} />
  }
}
```

### 包装组件

> 自定义渲染

- 丰富渲染
- 条件渲染
- 组合渲染

```js
import React from 'react';

function HOCStyleComponent (WrappedComponent, style) {
  return class WrappingComPonent extends React.Component {
    render () {
      return (
        <div style={style}>
          <WrappedComponent {...this.props} />
        </div>
      )
    }
  }
}
export default HOCStyleComponent;
```

## 高阶组件——继承

- 渲染劫持
- 操作生命周期
- 操作`props`
- 调用组件的静态方法

### 渲染劫持

```js
function MyHOC (WrapComponent) {
  return class extends WrapComponent {
    componentDidMount () {
      setTimeout(() => {
        console.log(this.props)
      }, 2000)
    }
    render () {
      let testRender = super.render();
      let newProps = { value: 'daimei' }
      let finalProps = Object.assign({}, testRender.props, newProps);
      let finalRender = React.cloneElement(
        testRender,
        finalProps,
        testRender.props.children
      );
      return finalRender
    }
  }
}
```

### 操作生命周期

控制render

```js
import React from 'react';

const HocComponent = (WrappedComponent) => {
  console.log(WrappedComponent.prototype.componentDidMount)
  console.log(WrappedComponent.prototype.render)
  const wrappedDidMount = WrappedComponent.prototype.componentDidMount;
  class MyContainer extends WrappedComponent {
    componentDidMount () {
      console.log('MyHoc componentDidMount')
      if (wrappedDidMount) {
        wrappedDidMount.apply(this) // 再调用一次
      }
    }
    render () {
      const eleTree = super.render();
      const newProps = Object.assign({}, eleTree.props);
      if (this.props.time && this.state.success) {
        return eleTree;
      }

      return <div>倒计时完成了...</div>
    }
  }
}

export default HocComponet;
```

## 示例

有时使用高阶组件，不能获取到真正包装的组件ref, 这时可以采用高阶组件解决这个问题

```js
/**
 * @name withRef
 * @func  getInstance
 * @description 处理父组件从子组件（包含HOC包裹的）获取ref实例
 * @example

 @withRef  // 注意：这句必须写在最接近`childComponent`的地方
 */

import React from 'react';

export default (WrappedComponent) => {
  return class withRef extends React.Component {
    static displayName = `withRef(${WrappedComponent.displayName || WrappedComponent.name || 'Component'})`;
    render () {
      // 这里重新定义一个props的原因是:
      // 你直接去修改this.props.ref在react开发模式下会报错，不允许你去修改
      const props = {
        ...this.props,
      };
      // 在这里把getInstance赋值给ref，
      // 传给`WrappedComponent`，这样就getInstance能获取到`WrappedComponent`实例
      props.ref = (el)=>{
        this.props.getInstance && this.props.getInstance(el);
        this.props.ref && this.props.ref(el);
      };
      return (
        <WrappedComponent {...props} />
      );
    }
  };
};
```
