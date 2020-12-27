### 前言

笔者认为，官方文档无疑是介绍最详细，表述最精准的参考。但对小白来说未免有点晦涩，本文意在以通俗、浅显的方式介绍 Vuex 的基本使用。

### 安装

```
npm i vuex -S
```

不同框架或平台，引入方式有些许不同：

- Vue CLI

```js
// main.js

import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```

- 360小程序

```json
// app.json

{
	"store": "store/index.js"
}
```