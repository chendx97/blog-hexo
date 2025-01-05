---
title: Vuepress2自定义主题详细指南
date: 2024-12-18 10:00:00
tags:
  - vuepress
categories:
  - 指南
cover: 
---

# 创建项目
首先，创建项目，`vuepress-starter`代表你的项目名，
```bash
npm init vuepress vuepress-starter
```
页面目录结构如下图所示，
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110195042.png)
然后启动项目，
```bash
npm run docs:dev
```
# 自定义首页
首先在`.vuepress/layouts/LayoutHome.vue`中定义首页如何布局，然后在`/vuepress/clients.js`中注册该布局，最后在`README.md`中声明首页使用此布局。
```js
// /vuepress/clients.js
import { defineClientConfig, resolvers  } from 'vuepress/client';
import LayoutHome from './layouts/LayoutHome.vue';

export default defineClientConfig({
  layouts: {
    LayoutHome,
  },
})
```
```
---
home: true
layout: LayoutHome
---
```
首页具体代码不展示。其他页面的自定义布局与首页不同，会在后面讲到。
# 自定义组件
组件都放在`.vuepress/components`下，一共有3种组件。
第一种，非全局组件，使用时需要引入，如下所示，
```
<script setup>
import ParentLayout from '@vuepress/theme-default/layouts/Layout.vue'
import ArticleList from '../components/ArticleList.vue'
</script>

<template>
  <ParentLayout>
    <template #navbar>
      hi
    </template>
    <template #page>
      <main class="page">
        <ArticleList :items="articles.items" />
      </main>
    </template>
  </ParentLayout>
</template>
```
第二种，全局组件需要注册，不需要引入，使用时才会被添加到页面中，如下所示，
```
// .vuepress/theme/client.js
import { defineClientConfig } from 'vuepress/client';
import Layout from './layouts/Layout.vue';
import Navbar from './components/Navbar.vue';

export default defineClientConfig({
  layouts: {
    Layout,
  },
  enhance({ app }) {
    app.component('Navbar', Navbar);
  },
});

```
第三种，rootComponents指定的组件会被直接放在根节点中，全局弹窗之类的组件可以用这种方法，
```
import { defineClientConfig } from 'vuepress/client'
import GlobalPopup from './components/GlobalPopup.vue'

export default defineClientConfig({
  rootComponents: [GlobalPopup],
})
```
# 自定义布局
在文件`.vuepress/layouts/LayoutDefault.vue`中定义页面内容。
```html
<!--.vuepress/layouts/LayoutDefault.vue-->
<script setup>
import Navbar from "../components/Navbar.vue";
import Footer from "../components/Footer.vue";
</script>
<template>
  <Navbar />
  <slot></slot>
  <Footer />
</template>
```
然后，在`.vuepress/client.js`中注册该布局，
```js
// .vuepress/client.js
import { defineClientConfig } from 'vuepress/client';
import LayoutDefault from './layouts/LayoutDefault.vue';
import Navbar from './components/Navbar.vue';

export default defineClientConfig({
  layouts: {
    LayoutDefault,
  },
});
```
最后在使用该布局的页面中，引入并将内容填充进slot。
```
<script setup>
import LayoutDefault from "./LayoutDefault.vue";
</script>
<template>
  <LayoutDefault>
    hi
  </LayoutDefault>
</template>
```
# 修改默认页面
默认的页面有文章列表页、标签页、类别页、时间轴页。有2种方案：替换默认插槽、替换整个布局页。
每个页面都有固定的文件名，如下所示：
- 文章列表页：layouts/Article.vue
- 文章内容页：layouts/Layout.vue
- 标签页：layouts/Tag.vue
- 类别页：layouts/Category.vue
- 时间轴页：layouts/Timeline.vue

下面以文章内容页为例，必须要在`.vuepress/layouts/Layout.vue`文件中定义文章内容页的内容。
方案一：替换默认插槽，
```
<!-- .vuepress/layouts/Layout.vue -->
<script setup>
import ParentLayout from "@vuepress/theme-default/layouts/Layout.vue";
</script>
<template>
  <ParentLayout>
    <template #navbar>
      <Navbar />
    </template>
    <template #page-bottom>
      <Footer />
    </template>
  </ParentLayout>
</template>
```
方案二：替换整个布局页，
```
<!-- .vuepress/layouts/Layout.vue -->
<script setup>
import LayoutDefault from "./LayoutDefault.vue";
</script>
<template>
  <LayoutDefault>
    <Content />
  </LayoutDefault>
</template>
```
`<Content/>`就是`.md`文件里的内容，不需要引入。

# 添加新页面
比如，想添加一个友链页。
需要先在`.vuepress/layouts/Friends.vue`中定义页面内容，然后在`.vuepress/client.js`中注册该布局。然后在`.vuepress/config.js`中添加新页面。
```js
// .vuepress/config.js
import { createPage } from 'vuepress/core'
export default defineUserConfig({
    // ...
    onInitialized: async (app) => {
        app.pages.push(await createPage(app, {
            path: '/friends.html',
            frontmatter: {
                layout: 'Friends',
            },
    }))
  }
})
```
# 如何覆盖默认样式
在`.vuepress/styles/index.scss`(一定是这个名字)中定义样式，不需要引入。
```css
// .vuepress/styles/index.scss
.text-color {
    color: red;
}
```
# 可能用到的api
## useClientData
```
<script setup lang="ts">
import { useClientData } from 'vuepress/client'

const {
  pageData,
  pageFrontmatter,
  pageHead,
  pageHeadTitle,
  pageLang,
  routeLocale,
  siteData,
  siteLocaleData,
} = useClientData()
</script>
```
`useClientData()`的返回值如下所示：
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110203033.png)
`useClientData()`返回全部客户端ref对象，另外还可以通过以下组合式api获取单独数据，
- usePageData 获取当前页面数据；
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110203042.png)
- usePageFrontmatter 获取当前页面的frontmatter配置；
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110203049.png)
- usePageHead 获取当前页面head信息，如meta、title等；
- ![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110203057.png)
- usePageHeadTitle 获取当前页面的title，页面title+站点title；
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110203105.png)
- usePageLang 获取当前页面语言；
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110203113.png)
## useRoutes
该api返回所有路由，
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241124201337.png)
## useSiteData
该api返回站点信息，
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241124201343.png)

## defineClientConfig
`defineClientConfig`需要定义在`.vuepress/client.js`中。如果是在自定义主题中使用，需要声明该文件的位置，
```js
// config.js
clientConfigFile: path.resolve(__dirname, './path/to/client.js'),
```
`defineClientConfig`的具体使用如下所示，
```js
import { defineClientConfig } from 'vuepress/client'

export default defineClientConfig({
  enhance({ app, router, siteData }) {
    // 可以在这里注册全局组件
    // app.component('MyComponent', MyComponent)
    
    // 可以使用路由方法
    router.beforeEach((to) => {
      console.log('before navigation')
    })
  },
  setup() {
    // 这里相当于根组件的setup的一部分，在这里可以使用vue的api
    // provide('count', 3)
  },
  layouts: {}, // 注册布局
  rootComponents: [], // 注册全局根组件
})
```
## resolveRoute
`useRoutes()`获取全部路由，是一个对象，key是path，值是路由信息；
`resolveRoutePath(url)`根据传入的url获取path；
`resolveRoute(link)`返回路由信息；
`withBase(path)` 拼接上base值；
在自定义主题时，可以通过`withBase`获取资源正确路径，
```js
<script setup>
import { ref } from 'vue'
import { withBase } from 'vuepress/client'

const logoPath = ref('/images/hero.png')
</script>

<template>
  <img :src="withBase(logoPath)" />
</template>
```
## 已定义的常量
- __VUEPRESS_VERSION__ vuepress核心包的版本；
- __VUEPRESS_BASE__ config.js中base配置；
- __VUEPRESS_DEV__ 是否是dev环境；
- __VUEPRESS_SSR__ 是否是ssr环境；
这些常量可以在`js`代码中直接使用，
```
<script setup>
console.log(__VUEPRESS_VERSION__)
</script>
```
## resolvers
覆盖一些默认的计算方法，实验性功能，修改时需充分了解原来的目的。
```js
import { defineClientConfig, resolvers } from 'vuepress/client'

export default defineClientConfig({
  enhance({ app, router, siteData }) {
    resolvers.resolvePageHeadTitle = (page, siteLocale) =>
      `${siteLocale.title} > ${page.title}`
  },
})
```
![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241110203057.png)
# 添加进阶功能
## 评论模块
首先，安装依赖，
```
npm i -D @vuepress/plugin-comment@next
```
然后，配置插件，申请评论服务，评论服务推荐`Giscus`，
https://ecosystem.vuejs.press/zh/plugins/blog/comment/giscus/
```js
// .vuepress/config.js
import { commentPlugin } from '@vuepress/plugin-comment'

export default defineUserConfig({
  plugins: [
    commentPlugin({
      provider: 'Giscus',
      repo: '...',
      repoId: '...',
      category: '...',
      categoryId: '...',
    }),
  ],
})
```
## @vuepress/helper辅助函数
https://ecosystem.vuejs.press/zh/tools/helper/

## 添加显示访问量功能；
https://ecosystem.vuejs.press/zh/plugins/analytics/umami-analytics.html


## 设置sitemap
https://vuepress-plugin-blog.billyyyyy3320.com/zh/guide/getting-started.html#frontmatter-%E5%88%86%E7%B1%BB%E5%99%A8
## 开启ssr
seo插件；https://ecosystem.vuejs.press/zh/plugins/seo/seo/

## 代码复制功能
https://ecosystem.vuejs.press/zh/plugins/features/copy-code.html
## 搜索功能
https://ecosystem.vuejs.press/zh/plugins/search/search.html#hotkeys


## 内置全局组件 
https://vuepress.github.io/zh/reference/components.html#autolink
## 默认插槽 
https://ecosystem.vuejs.press/zh/themes/default/extending.html#%E5%B8%83%E5%B1%80%E6%8F%92%E6%A7%BD
## 替换非全局组件 
https://ecosystem.vuejs.press/zh/themes/default/extending.html#%E7%BB%84%E4%BB%B6%E6%9B%BF%E6%8D%A2
## 非全局组件列表 
https://github.com/vuepress/ecosystem/tree/main/themes/theme-default/src/client/components
## 图片缩放功能 
https://ecosystem.vuejs.press/zh/plugins/features/medium-zoom.html
## 覆盖主题css变量
https://ecosystem.vuejs.press/zh/themes/default/styles.html#style-%E6%96%87%E4%BB%B6

# 最后
烂尾了，不一定补充。