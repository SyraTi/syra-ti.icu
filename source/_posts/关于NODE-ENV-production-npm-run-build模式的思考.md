---
title: 关于NODE_ENV=production && npm run build模式的思考
date: 2023-01-30 16:37:49
tags: 
- NodeJS
- NODE_ENV
- dependencies/devDependencies
---
## 前情提要
今天闲来无事，突然想起来之前在业务中看到的一个**“能跑，但是不太合乎逻辑”**的流程。
```shell
# 构建生产环境下的产物
NODE_ENV=production && npm install && npm run build
```
看起来一段非常简单的构建命令，实际会出现一些问题：当```NODE_ENV=production```时，```npm install```不会下载```devDependencies```，例如```webpack```、Typescript项目中的```@types/xxx```等。从而导致```npm run build```失败。

## 解决方案
**这时候可能有人会问了：我把```@types/xxx```丢到```dependencies```不就好了？**

话虽如此，但是这样真的正确吗？

在构建过程中```@types```会被用于编译js时的类型校验，```webpack```更不用说本身就用于构建，但是这些依赖在js产物中其实并不会用到。 所以它们理应存在于```devDependencies```而不是```dependencies```。

**那么在不更改```devDependencies```的情况下，这里得出的“能跑”的方案应该是：**
```shell
NODE_ENV=development && npm install && NODE_ENV=production && npm run build
```
确实问题得到了解决，```devDependencies```也被下载，不必担心```npm run build```过程中因为缺少依赖导致失败。

**至少，能跑了**

## 思考
问题是解决了，但是本着尽可能优雅的精神，能跑是能跑了，看到两遍```NODE_ENV```赋值，总有一些舍本逐末的感觉。下面是我的思考：
> 首先，理论上来讲，```build```确实是应该在dev环境下的行为，所以这时候下载```devDependencies```是合理的，那么运行```build```时，```NODE_ENV```理应为```development```。
> 但是，我们希望```build```命令所构建出的产物运行在生产环境下，那么，似乎```NODE_ENV```又应该是```production```。

于是歧义就产生了： **```build```过程中所使用的```NODE_ENV```，代表的的含义究竟是```build```命令运行时的环境，还是构建产物的环境？**

## 个人认为最优雅的方式
我的想法是：环境变量应当代表的是当前环境，而不是目标环境，所以```NODE_ENV```理应为```development```，但是各大开源库已经把```NODE_ENV=production && npm run build```这样的模式带成了惯例，导致了我们先入为主。
所以推翻这样的模式，采用额外的参数来代表目标环境更为合理：
```shell
NODE_ENV=development && npm install && npm run build --production
```
添加一个参数来决定构建目标，这样既明确了```install```和```build```都是dev环境下的行为，又确定了```build```的target是prod环境下的产物。 
个人觉得是最优雅的方式，也欢迎大家指点和讨论。
<br>

以上！まいど～

---
