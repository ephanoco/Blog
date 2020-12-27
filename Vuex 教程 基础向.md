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
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```

### refs

[官方文档](https://vuex.vuejs.org/zh/guide/)