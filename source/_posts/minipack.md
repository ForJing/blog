---
title: webpack打包原理分析
date: 2019-02-28 15:52:49
tags:
  - 前端
  - webpack
---

webpack 看起来很神奇， 其实它的打包原理就是读取文件， 分析文件，然后对代码做一些处理， 然后输出成新的文件。

首先我们建立如下的目录结构

![file](/minipack/webpack.png)
