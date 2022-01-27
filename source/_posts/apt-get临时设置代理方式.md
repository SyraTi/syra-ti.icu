---
title: apt-get临时设置代理方式
date: 2021-08-11 12:24:31
tags:
---

```
sudo apt-get -o Acquire::http::proxy="http://host:port/" install xxx