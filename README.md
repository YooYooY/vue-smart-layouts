# vue技巧-布局管理

整理自文章：[Vue tricks: smart layouts for VueJS](https://itnext.io/vue-tricks-smart-layouts-for-vuejs-5c61a472b69b)

[仓库地址](https://github.com/YooYooY/vue-smart-layouts)

## 前言

在写vue页面级整体布局时，一般的做法是在外层包裹布局组件：

```vue
<template>
  <MyLayout>
    <h1>Here is my page content</h1>
  </MyLayout>
</template>

<script>
import MyLayout from '@/layouts/MyLayout.vue'
export default {
  name: 'MyPage',
  components: { MyLayout }
}
</script>
```

这种做法会带来几个缺陷：

- 每个需要布局组件的页面都要引入布局组件（重复工作量）
- 内容需要被包裹在布局组件内（显得臃肿）
- 布局页面越多，代码越不好维护

## 初始化

利用 [vue-cli](https://cli.vuejs.org/) 初始化工程:
```sh
vue create vue-layouts
```
> vue2 和 vue3 的项目都行，后面都会讲到。


## 开始

整理src下的目录结构如下：
```
- src
    - views
        - About.vue
        - Contacts.vue
        - Home.vue
    - App.vue
    - main.js
    - router.js
```

- *App.vue*

```vue
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link> |
      <router-link to="/contacts">Contacts</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}
#nav a {
  font-weight: bold;
  color: #2c3e50;
}
#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
```

- *views/Home.vue*

```vue
<template>
  <div>
    <h1>This is a home page</h1>
  </div>
</template>

<script>
export default {
  name: 'Home'
}
</script>
```

- *views/About.vue*

```vue
<template>
  <div>
    <h1>This is an about page</h1>
  </div>
</template>

<script>
export default {
  name: 'About'
}
</script>
```

- *views/Contacts.vue*

```vue
<template>
  <div>
    <h1>This is a contacts page</h1>
  </div>
</template>

<script>
export default {
  name: "Contacts"
}
</script>
```

- *router.js*
```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue')
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue')
  },
  {
    path: '/contacts',
    name: 'Contacts',
    component: () => import('@/views/Contacts.vue')
  }
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router
```

接下来开始创建布局

## 核心布局组件

创建 *layouts/AppLayout.vue*，下面是 Vue2 的实现方式：

```vue
<template>
  <component :is="layout">
    <slot />
  </component>
</template>

<script>
const defaultLayout = 'AppLayoutDefault'
export default {
  name: "AppLayout",
  computed: {
    layout() {
      const layout = this.$route.meta.layout || defaultLayout
      return () => import(`@/layouts/${layout}.vue`)
    }
  }
}
</script>
```

这是一个非常简单的组件，确是整个布局组件的核心内容，首先创建动态组件`component`，根据计算属性`layout`决定加载返回的组件内容。

计算属性 `layout` 决定加载的组件由路由 `meta` 属性判断，如果不存在则加载默认布局组件，最后通过异步的方式加载布局组件。

下面是 Vue3 Composition API 的实现方式：

```vue
<template>
  <component :is="layout">
    <slot />
  </component>
</template>

<script>
import AppLayoutDefault from './AppLayoutDefault'
import { shallowRef, watch } from 'vue'
import { useRoute } from 'vue-router'

export default {
  name: 'AppLayout',
  setup () {
    const layout = shallowRef(AppLayoutDefault)
    const route = useRoute()
    watch(
      () => route.meta,
      async meta => {
        try {
          const component = await require(`@/layouts/${meta.layout}.vue`)
          layout.value = component.default || AppLayoutDefault
        } catch (e) {
          layout.value = AppLayoutDefault
        }
      },
    )
    return { layout }
  }
}
</script>
```

Vue3 的 `layout` 使用 `shallowRef` 保存对组件的引用，减少性能开销，同样 `markRaw` 也能达到效果。

## 创建页面布局

在创建页面布局组件之前，先对现有的代码做一些重构，首先是创建导航布局组件：

- *layouts/AppLayoutLinks*

```vue
<template>
  <div id="nav">
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link> |
    <router-link to="/contacts">Contacts</router-link>
  </div>
</template>

<script>
export default {
name: "AppLayoutLinks"
}
</script>

<style scoped>
#nav {
  padding: 30px;
}
#nav a {
  font-weight: bold;
  color: #2c3e50;
}
#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
```

- *App.vue*

```vue
<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
```

接下来创建`Home`、`About`、`Contacts` 和 默认页面的布局组件：

- *AppLayoutDefault.vue*

```vue
<template>
  <div>
    <slot />
  </div>
</template>
```

有一点需要注意的是，布局组件的命名需要跟声明动态组件的变量相同，组件的导入根据文件名查找：

```js
const defaultLayout = 'AppLayoutDefault'
...
const layout = this.$route.meta.layout || defaultLayout
return () => import(`@/layouts/${layout}.vue`)
```

- *AppLayoutHome.vue*

```vue
<template>
  <div>
    <header class="header" />
    <AppLayoutLinks />
    <slot />
  </div>
</template>

<script>
import AppLayoutLinks from '@/layouts/AppLayoutLinks'
export default {
  name: "AppLayoutHome",
  components: { AppLayoutLinks }
}
</script>

<style scoped>
.header {
  height: 5rem;
  background-color: green;
}
</style>
```

- *AppLayoutAbout.vue*
```vue
<template>
  <div>
    <header class="header" />
    <AppLayoutLinks />
    <slot />
  </div>
</template>

<script>
import AppLayoutLinks from '@/layouts/AppLayoutLinks'
export default {
  name: "AppLayoutAbout",
  components: { AppLayoutAbout }
}
</script>

<style scoped>
.header {
  height: 5rem;
  background-color: blue;
}
</style>
```

- *AppLayoutContacts.vue*
```vue
<template>
  <div>
    <header class="header" />
    <AppLayoutLinks />
    <slot />
  </div>
</template>

<script>
import AppLayoutLinks from '@/layouts/AppLayoutLinks'
export default {
  name: "AppLayoutContacts",
  components: { AppLayoutLinks }
}
</script>

<style scoped>
.header {
  height: 5rem;
  background-color: red;
}
</style>
```

这里为了演示，只是对不同布局组件的背景做了颜色变化。

## 在路由配置页面布局

在路由配置中，新增`meta`属性，配置不同页面的布局方式：

- *router.js*

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue'),
    meta: {
      layout: 'AppLayoutHome'
    }
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue'),
    meta: {
      layout: 'AppLayoutAbout'
    }
  },
  {
    path: '/contacts',
    name: 'Contacts',
    component: () => import('@/views/Contacts.vue'),
    meta: {
      layout: 'AppLayoutContacts'
    }
  }
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router
```

## 完成

最后将前面的工作串联起来，将` router-view` 包裹到高阶组件 `AppLayout`：

- *App.vue*
```vue
<template>
  <div id="app">
    <AppLayout>
      <router-view />
    </AppLayout>
  </div>
</template>
```

`AppLayout` 在 *main.js* 中注册成全局组件。

Vue2 版本的全局组件注册：
```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import AppLayout from '@/layouts/AppLayout'

Vue.component('AppLayout', AppLayout)

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

Vue3 版本的全局组件注册：

```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import AppLayout from './layouts/AppLayout'

createApp(App)
  .use(router)
  .component('AppLayout', AppLayout)
  .mount('#app')
```

## 总结

这种组件布局管理的方式在vue的应用中管理起来非常方便，主要有这几点便利：

- 轻松实现，在创建项目后花几分钟就可以调整好
- 使用简单，页面的布局管理在路由配置时就能设置，即使没有配置也会有默认布局
- 让组件看起来更干净，减少了布局组件的引入声明和高阶组件嵌套
- 把所有的布局管理逻辑放到了路由层而不是组件层

[仓库地址](https://github.com/YooYooY/vue-smart-layouts)