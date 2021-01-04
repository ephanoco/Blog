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
├── api
│   ├── login.js
│   └── ...
├── components
│   ├── Layout.vue
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

多个组件存在相同的依赖于 state 的计算属性，抽取到 getter 中，实现复用：

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
import { mapGetters } from 'vuex'

export default {
  computed: {
    ...mapGetters(['username']),
  },
}
</script>
```

- Mutation

通过提交 mutation 的方式更改状态：

```js
// store/mutation-types.js

// 使用常量替代 mutation 事件类型
export const SET_USER_INFO = 'SET_USER_INFO'
```

```js
// store/modules/login.js

import { SET_USER_INFO } from '../mutation-types'

const state = { ... }

const mutations = {
  // 第二个参数为额外参数（可选）
  [SET_USER_INFO](state, info) {
    state.userInfo = info
    setStorageSync('userInfo', info) // 登录态持久化
  },
}

export default {
  namespaced: true,
  state,
  mutations,
}
```

_tip: mutation 必须是同步函数_

```vue
<script>
import { mapMutations } from 'vuex'
import { SET_USER_INFO } from '../../store/mutation-types'

export default {
  methods: {
    ...mapMutations([
      `login/${SET_USER_INFO}`, // -> this[`login/${SET_USER_INFO}`](info)
    ]),
  },
}
</script>
```

- Action

```js
// store/modules/login.js

import { SET_USER_INFO } from '../mutation-types'
import * as api from '../../api/login'

const state = { ... }

const mutations = { ... }

const actions = {
  // 获取用户信息
  async getUserInfo({ state, commit }) {
    await api
      .getUserInfo({ ... })
      .then(response => {
        const { id, username, ... } = response.data
        commit(SET_USER_INFO, {
          id,
          username,
          // ...
        })
      })
  },
  // 注册全局 action
  getUserInfo: {
    root: true,
    async handler({ state, commit }) { ... },
  },
}

export default {
  namespaced: true,
  state,
  mutations,
  actions,
}
```

```js
// store/modules/pay.js

const actions = {
  // 查询是否支付成功
  async polling({ dispatch }) {
    // ...
    dispatch('getUserInfo', null, { root: true })
  },
}

export default {
  // ...
  actions,
}
```

```vue
// components/Layout.vue

<script>
import { mapActions } from 'vuex'

export default {
  created() {
    this.getUserInfo()
  },
  methods: {
    ...mapActions(['getUserInfo']),
  },
}
</script>
```

### ref

[官方文档](https://vuex.vuejs.org/zh/guide/)