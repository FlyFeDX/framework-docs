# 系统集成

## 微应用接入

我们最新采用了 micro-app 的方案。该方案没有沿袭 single-spa 的思路，而是借鉴了 WebComponent 的思想，通过 CustomElement 结合自定义的 ShadowDom，将微前端封装成一个类 WebComponent 组件，从而实现微前端的组件化渲染。并且由于自定义 ShadowDom 的隔离特性，micro-app 不需要像 single-spa 和 qiankun 一样要求子应用修改渲染逻辑并暴露出方法，也不需要修改 webpack 配置，是目前市面上接入微前端成本最低的方案。

在该方案中，框架应用只需要根据特定路由前缀渲染对应的子应用即可，这就要求接入的每个子应用的路由前缀全局唯一。

### 1. 子应用跨域支持。

- 开发环境

  ```js
  devServer: {
      headers: {
      'Access-Control-Allow-Origin': '*',
      }
  }
  ```

- 生产环境

  ```nginx
  location ^~ / {
      add_header Access-Control-Allow-Origin '*';
  }
  ```

### 2. 子应用设置部署路径唯一

- 在子应用 src 目录下创建名称为 public-path.js 的文件，并添加如下内容

  ```js
  // __MICRO_APP_ENVIRONMENT__和__MICRO_APP_PUBLIC_PATH__是由micro-app注入的全局变量
  if (window.__MICRO_APP_ENVIRONMENT__) {
    // eslint-disable-next-line
    __webpack_public_path__ = window.__MICRO_APP_PUBLIC_PATH__
  }
  ```

- 在子应用入口文件的最顶部引入 public-path.js

  ```js
  // entry
  import './public-path'
  ```

- 设置 webpack 配置中 publicPath 为唯一值。这里以/test 为例。

  ```js
  // webpack.config.js vue项目为对应的vue.config.js
  {
    output: {
      publicPath: '/test'
    }
  }
  ```

### 3. 子应用路由 basename 值与第二步中的唯一路径保持一致。

可以通过 window.**MICRO_APP_BASE_ROUTE**获取框架下发的 baseroute，如果没有设置 baseroute 属性，则此值默认为空字符串

react 需使用`MemoryRouter`

vue 需使用`mode: 'abstract'`模式

```js
// react
function render() {
  const root = createRoot(document.querySelector('#root'))
  root.render(
    <div className="web-app">
      {window.__MICRO_APP_ENVIRONMENT__ ? (
        // 👇 设置基础路由
        <MemoryRouter basename={window.__MICRO_APP_BASE_ROUTE__ || '/'}>
          <App />
        </MemoryRouter>
      ) : (
        <BrowserRouter basename={'/'}>
          <App />
        </BrowserRouter>
      )}
    </div>,
  )
}

// vue
const router = new VueRouter({
  // 👇 设置基础路由
  base: window.__MICRO_APP_BASE_ROUTE__ || '/',
  mode: window.__MICRO_APP_ENVIRONMENT__ ? 'abstract' : 'history',
  routes,
})
```

### 4. 路由跳转

框架会通过数据通信向子应用传递当前路由 `curRoute`,子应用需根据 curRoute 进行跳转

curRoute 中是带有 basename 前缀的 比如 /test/user,需手动切割 basename

```js
// react
useEffect(() => {
  if (curRoute) {
    history.push(curRoute.replace(window.__MICRO_APP_BASE_ROUTE__, ''))
  }
})

// vue
router.push({
  path: curRoute.replace(window.__MICRO_APP_BASE_ROUTE__, ''),
})
```

### 5. 监听卸载

子应用被卸载时会接受到一个名为 unmount 的事件，在此可以进行卸载相关操作。

```js
// react 入口文件
window.addEventListener('unmount', function () {
  ReactDOM.unmountComponentAtNode(document.getElementById('root'))
})
// vue 入口文件
window.addEventListener('unmount', function () {
  // app为vue实例
  app.$destroy()
})
```

## 数据通信

框架提供了一套灵活的数据通信机制，方便框架和子应用之间的数据传输。

框架会向子应用注入名称为`microApp`的全局对象，子应用通过这个对象和框架进行数据交互。

### 1. 子应用获取来自框架的数据

有两种方式获取来自框架的数据：

1. 直接获取数据

```js
if (window.__MICRO_APP_ENVIRONMENT__) {
  const data = window.microApp.getData() // 返回框架下发的data数据
}
```

2. 绑定监听函数

```js
function dataListener (data) {
  console.log('来自框架的数据', data)
}
window.microApp.addDataListener(dataListener, true)

/**
 * 绑定监听函数，监听函数只有在数据变化时才会触发
 * dataListener: 绑定函数
 * autoTrigger: 在初次绑定监听函数时如果有缓存数据，是否需要主动触发一次，默认为false
 * !!!重要说明: 因为子应用是异步渲染的，而框架发送数据是同步的，
 * 如果在子应用渲染结束前框架发送数据，则在绑定监听函数前数据已经发送，在初始化后不会触发绑定函数，
 * 但这个数据会放入缓存中，此时可以设置autoTrigger为true主动触发一次监听函数来获取数据。
 */
window.microApp.addDataListener(dataListener: Function, autoTrigger?: boolean)

// 解绑监听函数
window.microApp.removeDataListener(dataListener: Function)

// 清空当前子应用的所有绑定函数(全局数据函数除外)
window.microApp.clearDataListener()

```

### 2. 子应用向框架发送数据

```js
/**
 * dispatch只接受对象作为参数
 * type取自框架共享下来的messageTypes中的变量或者任意字符串
 * payload为需要传递的消息或者数据
 */
window.microApp.dispatch({
  type: messageTypes.openRoute,
  payload: { entry: '/' },
})
```

### 3. 框架共享数据详情

| 字段名       | 描述                                                                               |
| :----------- | :--------------------------------------------------------------------------------- |
| colorPrimary | 主颜色                                                                             |
| container    | 子应用的容器                                                                       |
| curRoute     | 要加载的路由                                                                       |
| mainTheme    | 整体风格 `'dark'` 或 `'light'`                                                     |
| module       | 模块名                                                                             |
| name         | 子应用唯一标识                                                                     |
| path         | 菜单中的路径                                                                       |
| src          | 子应用的完整 url                                                                   |
| systemInfo   | 系统信息 `{appId: 应用ID, appKey: 应用Key, messageTypes: 框架支持的消息交互类型,}` |
| userInfo     | 用户信息                                                                           |
| uuid         | uuid                                                                               |

`messageTypes`框架支持的消息交互类型

| 变量名               | 值                     |
| :------------------- | :--------------------- |
| changeTheme          | "changeTheme"          |
| closeAllTabs         | "closeAllTabs"         |
| closeCurrentTab      | "closeCurrentTab"      |
| closeLeftTabs        | "closeLeftTabs"        |
| closeOtherTabs       | "closeOtherTabs"       |
| closeRightTabs       | "closeRightTabs"       |
| closeTabs            | "CloseTabs"            |
| frameworkLoading     | "frameworkLoading"     |
| logout               | "logout"               |
| modifyTabDisplayName | "modifyTabDisplayName" |
| openRoute            | "OpenRoute"            |
| recordOnline         | "recordOnline"         |
| refreshAllTabs       | "refreshAllTabs"       |
| refreshTab           | "refreshTab"           |
| selectTabs           | "SelectTabs"           |
| switchUser           | "switchUser"           |
