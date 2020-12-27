### 前言

笔者认为，官方文档无疑是介绍最详细，表述最精准的参考。但对小白来说未免有点晦涩，本文意在以通俗、浅显的方式介绍 Vuex 的基本使用。

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

### 项目结构

```
├── // root
└── store
    ├── modules
    │   ├── login.js      # 登录模块
    │   └── pay.js        # 支付模块
    ├── getters.js
    ├── index.js          # entry
    └── mutation-types.js
```

```js
// index.js

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

### refs

[官方文档](https://vuex.vuejs.org/zh/guide/)