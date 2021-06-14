# Vue路由学习笔记

### 入门

**html**

```html
<script src='/vue-router/js/vue.js'></script>
<script src='/vue-router/js/vue-router.js'></script>
<div id="app">
    <h1>hello app</h1>
    <p>
        <!-- 路由链接 -->
        <router-link to="/foo">Go to foo</router-link>
        <router-link to="/bar">Go to Bar</router-link>
    </p>
    <!-- 跳转后路由在此处渲染 -->
    <router-view></router-view>
</div>
```

**JS**

```js
// 1.定义路由组件
const foo = {template:'<div>foo</div>'}
const bar = {template:'<div>bar</div>'}
// 2.定义一些路由，每个路径映射到一个组件
const routes = [
    {path:'/foo',component:foo},
    {path:'/bar',component:bar}
]
// 3.创建路由实例，导入路由选项
const router = new VueRouter({
    routes:routes
})
// 4.创建并挂载到根实例
const app = new Vue({
    router
}).$mount('#app')
```

### 1. 动态路由匹配

将给定模式的路由匹配到相同组件。可能有一个`User`组件，它应该为所有用户呈现，但具有不同的用户 ID。

```js
const router = new VueRouter({
  routes: [
    // dynamic segments start with a colon
    { path: '/user/:id', component: User }
  ]
})
```

此时，`/user/bar`和`/user/foo`都将映射到组件User。动态段由冒号表示`:`，同一条路径可以有多个动态段。

```js
const routes = [
    {path:'/user/:id/post/:postId',component:User}
]
```

#### 1.1 响应参数更改

使用带有参数的路由时，将进行组件的重用。这一过程不进行组件的销毁和创建，所以不会触发相应的钩子函数。

- 要对同一组件的`params`更改作出反应，可以通过观察`$params`对象：

```js
const User = {
  template: '...',
  watch: {
    $route(to, from) {
      // react to route changes...
    }
  }
}
```

- 或使用`$beforeRouteUpdate`导航守卫钩子：

```js
const User = {
  template: '...',
  beforeRouteUpdate(to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

#### 1.2 Catch all或404 not found Route

```js
{
  // will match everything
  path: '*'
}
{
  // will match anything starting with `/user-`
  path: '/user-*'
}
```

使用星号时，参数`pathMatch`会自动添加到`$route.params`. 它包含与星号匹配的 url 的其余部分：

```js
// Given a route { path: '/user-*' }
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'

// Given a route { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```

该路由`{ path: '*' }`通常用于 404 客户端。

#### 1.3 高级匹配模式

[高级匹配模式文档](https://github.com/pillarjs/path-to-regexp/tree/v1.7.0#parameters)

#### 1.4 匹配优先级

有时同一个 URL 可能会被多个路由匹配。在这种情况下，匹配的优先级由路由定义的顺序决定：越早定义路由，它获得的优先级越高。

### 2. 嵌套路由

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id',
      component: User,
      children: [
        {
          path: 'profile',
          component: UserProfile
        },
        {
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

当访问`/user/foo/profile`时，对应组件`UserProfile`,当访问`/user/foo/posts`时，对应组件`UserPosts`。

如果给children中的path加`/`，则表示根路径：

```js
children: [
    {
        path: '/profile',
        component: UserProfile
    },
    {
        path: '/posts',
        component: UserPosts
    }
]
```

只有访问`/posts`才能访问到组件`UserPosts`。

### 3. 程序化导航

#### 3.1 router.push

除了`<router-link>`用于创建用于声明性导航的锚标记之外，我们还可以使用路由器的实例方法以编程方式完成此操作。

```js
router.push(location, onComplete?, onAbort?);
```

此方法将一个新条目推送到历史堆栈中，因此当用户单击浏览器后退按钮时，他们将被带到上一个 URL。

单击`<router-link :to="...">`相当于调用`router.push(...)`。

参数可以是字符串路径或位置描述符对象：

```js
// literal string path
router.push('home')

// object
router.push({ path: 'home' })

// named route
router.push({ name: 'user', params: { userId: '123' } })

// with query, resulting in /register?plan=private
router.push({ path: 'register', query: { plan: 'private' } })
```

如果提供了`path`则，`params`不起作用，而`query`起作用。

```js
const userId = '123'
router.push({ name: 'user', params: { userId } }) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// This will NOT work
router.push({ path: '/user', params: { userId } }) // -> /user
```

相同的规则适用于组件的`to`属性`router-link`。

可选择提供`onComplete`和`onAbort`回调在`router.push`或`router.replace`中作为第二个和第三个参数。这些回调将分别在导航成功完成（在所有异步钩子解决后）或中止（在当前导航完成之前导航到同一路线或不同路线）时调用。

#### 3.2 router.replace

它的作用就像`router.push`，唯一的区别是它在导航时不会推送新的历史条目，正如其名称所暗示的那样 - 它替换了当前条目。

相当于`<router-link :to="..." replace>`

#### 3.3 router.go(n)

以整数作为参数，指示在历史堆栈中前进或后退的步数，类似于`window.history.go(n)`。

```js
// go forward by one record, the same as history.forward()
router.go(1)
window.history.forward()

// go back by one record, the same as history.back()
router.go(-1)
window.history.back()
// go forward by 3 records
router.go(3)

// fails silently if there aren't that many records.
router.go(-100)
router.go(100)
```

> `router.push`、`router.replace`和`router.go`是`window.history.pushState`、`window.history.replaceState`和`window.history.go`

### 4. 命名路线

可以`routes`在创建 Router 实例时在选项中为路由命名：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

要链接到命名路由，您可以将对象传递给`router-link`组件的`to`prop：

```html
<router-link to="{name:'user',params:{userId:123}}"></router-link>
```

或以编程方式：

```js
router.push({name:'user',params:{userId:123}});
```

在这两种情况下，路由器都会导航到路径 `/user/123`。

### 5. 命名视图

可以拥有多个视图，并为它们命名：

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

没有名字的，`dedault`是它的名字。

视图是使用组件渲染的，因此多个视图需要多个组件才能用于同一路由。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

#### 5.1 嵌套命名路由

