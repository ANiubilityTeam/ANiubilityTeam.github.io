---
title: Vue3.0 + Vite + TS + ElementPlus项目搭建
author: RemindAutumn
tags: 
  - Vue
  - Vite
  - TS
  - ElementPlus
categories: 
  - 前端工程化
description: Vue3.0 + Vite + TS + ElementPlus项目搭建
date: 2021-06-19
---
# Season One （Vue3.0 + Vite + TS + ElementPlus）

## 前置环境

* **系统**  
windows

* **node**  
版本 14.17.0

* **yarn**  
版本 1.22.10

> Vite 需要 Node.js 版本 >= 12.0.0。

---

## 一、创建项目

### 相关命令

```sh
    yarn create @vitejs/app demo
    cd demo
    yarn
    yarn dev
```

### 创建并选择配置

使用 `yarn create @vitejs/app demo` 命令，会出现两次选择提示, 我们依次选择vue，vue-ts即可。

```sh
» - Use arrow-keys. Return to submit.
    vanilla
>   vue
    react
    preact
    lit-element
    svelte
```

```sh
» - Use arrow-keys. Return to submit.
    vue
>   vue-ts
```

### 添加 `package.json` 配置

在项目创建成功，目录生成后，找到 `package.json` 文件。添加 `"license": "ISC"` ，和 `--host` 。

```json
{
  "license": "ISC", //添加此行。开源许可证
  "name": "bone",
  "version": "0.0.0",
  "scripts": {
    "dev": "vite --host", //添加 --host 开启公开访问.
    "build": "vue-tsc --noEmit && vite build",
    "serve": "vite preview"
  },
  "dependencies": {
    "vue": "^3.0.5",
    "vue-router": "4"
  },
  "devDependencies": {
    "@types/node": "^15.6.1",
    "@vitejs/plugin-vue": "^1.2.2",
    "@vue/compiler-sfc": "^3.0.5",
    "element-plus": "^1.0.2-beta.45",
    "sass": "^1.34.0",
    "typescript": "^4.1.3",
    "vite": "^2.3.4",
    "vue-tsc": "^0.0.24"
  }
}
```

### 安装 `node_modules` 并运行项目

在 `yarn dev` 成功后，在终端应该可以看到如下内容

```sh
  vite v2.3.7 dev server running at:

  > Network:  http://192.168.1.44:3000/
  > Local:    http://localhost:3001/

  ready in 351ms.
```

## 二、使用vue-router

因为使用的vite默认时没有vue-router的, 使用 yarn add vue-router@4 安装即可

### vue-router的安装命令

```sh
    yarn add vue-router@4
```

### type/node安装命令

因为使用别名的时候需用用到 node ， 而这是ts 项目，如果没有安装 `type/node` 的话， 此时也需要安装一下。

```sh
    yarn add @types/node -D
```

### 别名配置

安装完成后， 先不急着去新建*路由*文件 。先修改一下 `vite.config.ts` 文件 , 配置**别名** ，和**webpack**类似，**vite**也有个 `alias` 属性。

```ts
    /* vite.config.ts */
    import { defineConfig } from 'vite'
    import vue from '@vitejs/plugin-vue'
    const { resolve } = require("path");//引入resolve

    export default defineConfig({
        plugins: [vue()],

        resolve: { // resolve相关配置
            alias: { //别名配置
                "@": resolve(__dirname, "/src"), //配置别名 
            },
        },

    })
```

> 如果没有安装 `type/node` ，那么 `const { resolve } = require("path")` 会报错嗷， 安装一下即可。

### 路径配置

在我们配置完别名属性之后，因为我们使用ts，为了避免一会开发, 编写路由文件时劈里啪啦的错误提示和警告，我们也需要去配置一下 `tsconfig.json` .

先看一下此时默认的 `tsconfig.json` 文件的配置. 让我们打上简单的注释

```json
/* 详细配置请参考官网 ✈✈ https://www.tslang.cn/docs/handbook/compiler-options.html */
{
  "compilerOptions": {
    "target": "esnext", //指定ECMAScript目标版本 es5 这些, 此处的esnext 是最新版本的意思下同
    "module": "esnext", // 指定生成哪个模块系统代码
    "moduleResolution": "node",// 决定如何处理模块。或者是"Node"对于Node.js/io.js，或者是"Classic"（默认）。 
    "strict": true,// 启用所有严格类型检查选项。
    "jsx": "preserve", // 支持JSX
    "sourceMap": true, // 是否生成相应的 .map文件
    "resolveJsonModule": true, // 这个我居然没在配置里查到 ,离谱 💢 根据字面意思 可能是是否导出json 模块?
    "esModuleInterop": true, // 允许从没有设置默认导出的模块中默认导入。这并不影响代码的输出，仅为了类型检查。
    "lib": ["esnext", "dom"],  //编译过程中需要引入的库文件的列表 这个自己取看文档,上面有小飞机.
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"]
}

```

然后我们需要加上关于 path 的配置. 告诉 ts , 对应的地址别名 指向的文件地址. 避免在ts中使用别名的时候, 报错"找不到文件".

```json
  {
    "compilerOptions": {
      /* ... */
      "baseUrl": "", // 🦴路径映射 , 注意 配置 paths 的时候如果不设置baseUrl 会报错嗷.
      "paths": { //这边就是路配置拉, 配置了这个就可以自动补全地址拉.
        "@/*": ["src/*"],
      }
      /* ... */
    },
  }
```

配置完成后, 我们就可以开始配置我们第一个路由地址文件了。

### 新建router文件

此时我们就可以在 `src` 下新建一个 `router` 文件夹, 然后新建 `index.ts` 文件.(见目录)

在v3.0 中  vue router 的使用方式和vue2.0 并没有巨大的用法区别.

```ts
/* src/router/index.ts */

/* 首先我们需要先引入`vue-router`提供给我们的创建方法 */
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'

import Lay from '@/views/Layout'
const routes: Array<RouteRecordRaw> = [
    {
        path: '/',
        component: Lay,
        children: [
            {
                path: 'home',
                name: 'home',
                component: () => import('@/views/Home/index')
            }
        ]

    },
    {
        path: '/hellow',
        name: 'TodoList',
        component: () => import('@/components/HelloWorld.vue')
    },
]

const router = createRouter({
    history: createWebHistory(), // 路由模式 一样的 有hash 模式还有history模式,此处使用的时 history 模式
    routes,            
})
export default router
```

包括在main.ts 中引入 .  

下一期介绍vue3种的一些基本写法.在v3中v2对应的一些写法.
