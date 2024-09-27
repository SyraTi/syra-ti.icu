---
title: 添加了~/.bash_profile之后，~/.bashrc失效以及bash失去高亮的原因及解决方案
date: 2021-09-29 12:52:27
tags:
---

## 问题表现
添加```~/.bash_profile```之后，bash失去高亮，```~/.bashrc```中的逻辑不运行。
![添加前](../images/posts/b49d241eeab14f43b53edb783eb9d8f1.png)
添加前↑
![添加后](../images/posts/fd0ce34e0824462fb70387c80a53c57f.png)
添加后↑
<!-- more -->
## 问题原因
在添加```~/.bash_profile```之后，```~/.profile```会被覆盖不再运行，而load bashrc的逻辑在```~/.profile```中，导致```~/.bashrc```也一并失效。
![~/.profile中加载bashrc的逻辑](../images/posts/50f8cda872f34b57a3f9f7e7b3d1f48c.png)
```~/.profile```中加载```~/.bashrc```的逻辑↑

## 解决方案
- 方案一：将```~/.bash_profile```中的逻辑移至```~/.profile```，并删除```~/.bash_profile```。
- 方案二：在```~/.bash_profile```中添加以下代码保证```~/.profile```中的代码运行。
 ```bash
 source ~/.profile
 ```


<br>
以上！まいど～

---------
