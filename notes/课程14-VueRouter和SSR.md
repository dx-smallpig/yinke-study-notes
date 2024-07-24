# Vue Router & SSR

## 一、Vue Router 基础使用

最新的`Vue Router`已经发展到了 V4 版本。

1. 从发展趋势来看，后续的前端路由都会往函数式的编程方式发展（如`useRouter`, `useRoute`）。不管是`React Router` 还是 `Vue Router`，也都趋向于`Hooks`的使用。

路由的目的：将我们的组件映射到路由上，让` Vue Router` 知道在哪里渲染它们。

```vue
<script src="https://unpkg.com/vue@3"></script>
<script src="https://unpkg.com/vue-router@4"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!--使用 router-link 组件进行导航 -->
    <!--通过传递 `to` 来指定链接 -->
    <!--`<router-link>` 将呈现一个带有正确 `href` 属性的 `<a>` 标签-->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/about">Go to About</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

其中：

- `router-link` 类似 `a` 标签，这使得 `Vue Router` 可以在不重新加载页面的情况下更改 URL，处理 URL 的生成以及编码；
- `router-view` 将显示与 url 对应的组件；

一个最基本的例子：

```JavaScript
// 1. 定义路由组件.
// 也可以从其他文件导入
const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

// 2. 定义一些路由
// 每个路由都需要映射到一个组件。
// 我们后面再讨论嵌套路由。
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 3. 创建路由实例并传递 `routes` 配置
// 你可以在这里输入更多的配置，但我们在这里
// 暂时保持简单
const router = VueRouter.createRouter({
  // 4. 内部提供了 history 模式的实现。为了简单起见，我们在这里使用 hash 模式。
  history: VueRouter.createWebHashHistory(),
  routes, // `routes: routes` 的缩写
})

// 5. 创建并挂载根实例
const app = Vue.createApp({})
//确保 _use_ 路由实例使
//整个应用支持路由。
app.use(router)

app.mount('#app')

// 现在，应用已经启动了！
```























































































































































































































































