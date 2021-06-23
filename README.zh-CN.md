# vite-plugin-externals

[English](README.md) | 简体中文

使用外部库，类似webpack的externals，但现在只支持浏览器环境。

可以不另外配置 `rollup` 的选项，就可以使用在生产环境。

但是在默认配置下在 `commonjs` 环境中不生效，例如 `ssr`。

## 用法

```bash
npm i vite-plugin-externals -D
```

增加配置在 `vite.config.js`

```js
// vite.config.js
import { viteExternalsPlugin } from 'vite-plugin-externals'

export default {
  plugins: [
    viteExternalsPlugin({
      vue: 'Vue',
      react: 'React',
      'react-dom': 'ReactDOM',
    }),
  ]
}
```
## 原理

转换js的源代码。

```js
// 选项
viteExternalsPlugin({
  vue: 'Vue',
}),
// 源代码
import Vue from 'vue'
// 转换后
const Vue = window['Vue']

// 源代码
import { reactive, ref as r } from 'vue'
// 转换后
const reactive = window['Vue'].reactive
const r = window['Vue'].ref
```

**注意**: 请使用该插件前，需要把代码转换成js，因为此插件只能解析js代码，例如：

```js
import vue from '@vitejs/plugin-vue'

export default {
  plugins: [
    vue(), // @vitejs/plugin-vue 将会把SFC的代码转换成js代码

    // 所以这插件要放在@vitejs/plugin-vue的下面
    viteExternalsPlugin({
      vue: 'Vue',
    }),
  ]
}
```

## 配置选项

### filter

此插件会默认过滤 `node_modules` 下的文件，只转换 js/ts/vue/jsx/tsx 文件。

你可以指定 `filter` 函数，返回 `true` 将会进行转换成外部变量。

```js
viteExternalsPlugin({
  vue: 'Vue',
}, {
  filter(code, id, ssr) {
    // 你的代码
    return false
  }
}),
```

### useWindow

默认为 `true` , 设置 `false` , `window` 的作用域将不会加上。

**注意**: 如果模块名有特殊字符，例如 `/`，设置useWindow选项 `false` 将引发错误。

```js
viteExternalsPlugin({
  vue: 'Vue',
}, { useWindow: false }),

// 源代码
import Vue from 'vue'
// 转换后, 不是 `const Vue = window['Vue']`
const Vue = Vue
```