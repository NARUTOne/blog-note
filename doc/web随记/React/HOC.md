# 高阶组件 HOC

> 高阶组件就是一个接收一个组件并返回另外一个新组件的函数！

## 代理高阶

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

## 继承高阶

- 操作生命周期
- 操作`props`

### 操作生命周期

控制render

```js
import React from 'react';

const HocComponent = (WrappedComponent) => {
  class MyContainer extends WrappedComponent {
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