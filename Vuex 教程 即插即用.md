### 前言

笔者认为，官方文档无疑是介绍最详细，表述最精准的参考。但对小白来说未免有点晦涩，本文意在以更直观的方式帮助开发者快速上手 Vuex。

### 安装

```
npm i vuex -S
```

### 引入

```js
// main.js

import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```

_场景：用户登录 => 获取用户信息 userInfo => 多个组件共享_

### 项目结构

```
├── components
│   ├── Sidebar.vue
│   └── ...
└── store
    ├── modules
    │   ├── login.js      # 登录模块
    │   └── pay.js        # 支付模块
    ├── getters.js
    ├── index.js          # entry
    └── mutation-types.js
```

```js
// store/index.js

import getters from './getters'

// 自动引入 modules 目录下的模块 ref: vue-element-admin
// https://webpack.js.org/guides/dependency-management/#requirecontext
const modulesFiles = require.context('./modules', true, /\.js$/)

// you do not need `import app from './modules/app'`
// it will auto require all vuex module from modules file
const modules = modulesFiles.keys().reduce((modules, modulePath) => {
  // set './app.js' => 'app'
  const moduleName = modulePath.replace(/^\.\/(.*)\.\w+$/, '$1')
  const value = modulesFiles(modulePath)
  modules[moduleName] = value.default
  return modules
}, {})

export default {
  modules,
  getters,
}
```

### 核心概念

- State

```js
// store/modules/login.js

const state = {
  // 调用 getStorageSync 实现登录态持久化
  userInfo: getStorageSync('userInfo') || {
    id: '', // 用户id
    username: '', // 用户名称
    // ...
  },
}

export default {
  namespaced: true, // 命名空间
  state,
}
```

```vue
// components/Sidebar.vue

<template>
  <div v-text="userInfo.id" />
</template>

<script>
// 引入 mapState 辅助函数
import { mapState } from 'vuex'

export default {
  computed: {
    // 映射 this.userInfo 为 store.state.login.userInfo
    ...mapState('login', ['userInfo']),
  },
}
</script>
```

- Getter

多个组件存在相同的依赖于 state 的计算属性，避免冗余，将其抽离到 getter 中：

```js
// store/getters.js

const getters = {
  username: state => {
    const { username } = state.login.userInfo
    return username.length > 6 ? `${username.substr(0, 6)}...` : username
	},
}

export default getters
```

```vue
// components/Sidebar.vue

<template>
  <div v-text="username" />
</template>

<script>
// 引入 mapGetters 辅助函数
import { mapGetters } from 'vuex'

export default {
  computed: {
    ...mapGetters(['username']),
  },
}
</script>
```

### ref

[官方文档](https://vuex.vuejs.org/zh/guide/)