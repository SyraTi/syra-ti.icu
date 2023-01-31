---
title: 关于vue-cli4无法配置自定义devtool选项的问题
date: 2021-10-26 14:18:33
tags:
---

## 问题表现
在```vue.config.js```中无法配置自定义```devtool```选项，无论是使用```configureWebpack```还是```chainWebpack```都无法改变```source-map```的类型。官方文档也没有提供相关的说明。

```javascript
// vue.config.js
const Components = require('unplugin-vue-components/webpack')
const { ElementPlusResolver } = require('unplugin-vue-components/resolvers')
// 两种方式均无法更改devtool
module.exports = {
  configureWebpack: {
    devtool: "source-map",
    // 2021.10.26更新 ↓罪魁祸首
    plugins: [
      Components({
        resolvers: [ElementPlusResolver()]
      })
    ]
  },
  chainWebpack: config => {
    config
      .devtool('source-map')
  }
}
```
<!-- more -->
## 问题原因
**2021.10.26更新**： 问题产生的原因不是```vue-cli```，而是```element-ui```提供的按需加载插件```unplugin-vue-components/resolvers```破坏了sourcemap，具体的原因不清楚，似乎插件作者也不是很有```sourcemap```方面的开发意愿。 总之遇到这个问题的话看一看有没有这个玩应吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e7165c44fff043758752a54008586582.png)

---
### 本来下面这里写了一大通，最后发现不是这个原因，错怪尤老师了，对不起x 下面的都可以不用看了
~~```vue-cli```在```npm run serve```时强制使用了```'eval-cheap-module-source-map'```~~

```javascript
// @vue/cli-service/lib/commands/serve.js
// line: 51
 api.chainWebpack(webpackConfig => {
      if (process.env.NODE_ENV !== 'production' && process.env.NODE_ENV !== 'test') {
       // 注意这里：在npm run serve时强制使用了'eval-cheap-module-source-map'
        webpackConfig
          .devtool('eval-cheap-module-source-map')

        webpackConfig
          .plugin('hmr')
            .use(require('webpack/lib/HotModuleReplacementPlugin'))

        // https://github.com/webpack/webpack/issues/6642
        // https://github.com/vuejs/vue-cli/issues/3539
        webpackConfig
          .output
            .globalObject(`(typeof self !== 'undefined' ? self : this)`)

        if (!process.env.VUE_CLI_TEST && options.devServer.progress !== false) {
          webpackConfig
            .plugin('progress')
            .use(require('webpack/lib/ProgressPlugin'))
        }
      }
    })
```

~~截止目前（2021年10月26日），```vue-cli```的最新```realese```版本（v4.5.14）中，依旧存在这个问题。但是github的开源代码的```dev```分支已经修复此问题（[查看代码](https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-service/lib/commands/serve.js)），不知道什么时候会发布到正式版本中去。~~

## ~~解决方案~~
~~**方案一：** 手动修改```node_modules/@vue/cli-service/lib/commands/serve.js:54```， 将```'eval-cheap-module-source-map'```改为想要的模式。弊端是install之后可能需要再改一下，如果不在意```source-map```的话还是权衡一下利弊。~~

~~**方案二：** 使用```vue-cli```的```dev```分支源代码编译一份最新的来使用。~~



<br>

以上！まいど～

---
