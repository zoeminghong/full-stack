# Taro 学习笔记

[Sass学习资料](https://sass.bootcss.com/guide)

## Taro 页面

Taro.navgateTo 跳转到有返回按钮的页面

Taro.redirectTo 跳转到一个页面

删除 state 值

```js
delete this.state.username
```

设置 state 值

```js
this.setState({
	username:'ss'
})
```

view 组件

Taro.getApp(Object)：获得App 实例方式

this.$router.params 获取路由参数

## Taro 后端交互

[Taro.request(OBJECT)](https://nervjs.github.io/taro/docs/apis/network/request/request.html#docsNav)

## Taro UI

https://taro-ui.jd.com/#/docs/icon

## Taro 项目示例

https://github.com/btc022003/like-content-wechat-app