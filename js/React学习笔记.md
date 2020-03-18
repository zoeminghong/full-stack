# React学习笔记

#### 对象

中括号，或者点

#### 路由

```js
dev: {

    // Paths
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {
      '/api': {
        target: 'http://XX.XX.XX.XX:8083',
        changeOrigin: true,
        pathRewrite: {
          '^/api': '/api'   // 这种接口配置出来     http://XX.XX.XX.XX:8083/api/login
          //'^/api': '/' 这种接口配置出来     http://XX.XX.XX.XX:8083/login
        }
      }
    }
  },
```

> 域名一定要加http或者https https://www.webpackjs.com/configuration/dev-server/#devserver-proxy

[getFieldDecorator用法（一）——登录表单](https://www.cnblogs.com/mosquito18/p/9754192.html)

### State

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Get

```js
this.state.squares
```

Set

```js
this.setState({squares: squares});

this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

### Props

```js
  renderSquare(i) {
    return (
      <Square
        value={this.state.squares[i]}
        onClick={() => this.handleClick(i)}
      />
    );
  }
  
  ------
  // Square 组件下
  this.props.value
	this.props.onClick
```

### 组件

#### 函数组件

如果你想写的组件只包含一个 `render` 方法，并且不包含 state，那么使用**函数组件**就会更简单。我们不需要定义一个继承于 `React.Component` 的类，我们可以定义一个函数，这个函数接收 `props` 作为参数，然后返回需要渲染的元素。函数组件写起来并不像 class 组件那么繁琐，很多组件都可以使用函数组件来写。

```js
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}
```

##### 渲染

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

#### 类组件

```js
class Board extends React.Component {

}
```

### 事件绑定

```js
//方法一
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // 为了在回调中使用 `this`，这个绑定是必不可少的
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

```js
// 方法二 
render() {
    return (
      <button onClick={this.handleClick.bind(this)}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
 }
```

```js
// 方法三
class LoggingButton extends React.Component {
  // 此语法确保 `handleClick` 内的 `this` 已被绑定。
  // 注意: 这是 *实验性* 语法。
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

```js
// 方法四
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```

#### 值传递

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

在这两种情况下，React 的事件对象 `e` 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显式的进行传递，而通过 `bind` 的方式，事件对象以及更多的参数将会被隐式的进行传递。

### key

key 帮助 React 识别哪些元素改变了，比如被添加或删除。因此你应当给数组中的**每一个元素赋予一个确定的标识**。它们不需要是全局唯一的，只需要兄弟节点唯一。key 会传递信息给 React ，但不会传递给你的组件。

```js
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

### 组件聚焦

DOM Refs

https://zh-hans.reactjs.org/docs/accessibility.html

### 事件

https://zh-hans.reactjs.org/docs/accessibility.html

getFieldDecorator

https://www.cnblogs.com/mosquito18/p/9754192.html



class 下面的引用要加this，不能用const

方法下面不用this，要加const

### createUse 与 useRef 区别

createRef 每次渲染都会返回一个新的引用，而 useRef 每次都会返回相同的引用。

https://blog.csdn.net/frontend_frank/article/details/104243286

### Dva

https://www.cnblogs.com/lucas27/p/9292058.html

https://blog.csdn.net/weixin_42442713/article/details/91351890

## 参考文档

[JavaScript 6](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript)

[ProTable](https://www.jianshu.com/p/98e3a043505e)



