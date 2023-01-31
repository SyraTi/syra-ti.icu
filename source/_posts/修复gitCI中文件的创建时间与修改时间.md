---
title: 修复gitCI中文件的创建时间与修改时间
date: 2022-04-11 00:06:43
tags: CI/CD
---

# 前情提要
hexo发现经过CI之后，所有文章的 **更新时间(mtime)** 变成了CI运行的时间。

# 解决思路
在CI checkout代码时，使用文件最后一次涉及的commit时间作为文件的修改时间，进行mtime的恢复。

<!-- more -->
# 解决方案
```yml
# .github/workflows/action.yml
...
- uses: actions/checkout@v3
  # 如果使用的是GithubAction的actions/checkout，记得加上下面的内容，对仓库历史进行完整的签出
  with:
    fetch-depth: 0
# 恢复文件修改时间
- name: Restore file modification time
  run: |
    find source/_posts -name '*.md' | while read file; do touch -d "$(git log -1 --pretty="@%ct" -- $file)" $file; done
...
```

# 代码解释
```shell
# 列出需要恢复修改时间的文件
find source/_posts -name '*.md'
# 循环
# 使用 git log -- $file 得到该文件最后修改的commit 并且使用pretty选项获取时间戳
# 使用 touch -d 来将文件的atime以及mtime修改为commit的时间戳
while read file; do touch -d "$(git log -1 --pretty="@%ct" -- $file)" $file; done
```


以上！まいど～

---
