---
title: Electron编译/打包流程
tags: []
id: '75'
categories:
  - - 软件工程
date: 2020-03-14 23:21:49
---

写完一个electron项目后准备编译测试一下，结果遇到重重困难，一堆ERROR看的头疼，这里记录一下解决过程，方便看到这篇文章的人避免踩坑。

总所周知由于国情的原因，大部分开发者用npm都换成了淘宝源

```
npm config set registry https://registry.npm.taobao.org
```

（咕咕咕）