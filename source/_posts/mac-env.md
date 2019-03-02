---
title: 使用环境变量避免暴露敏感信息
date: 2019-03-02 13:51:07
tags:
---

使用 vim 在 `~/.bash_profile` 加入

```
export mongodb_url=mongodb+srv://<username>:<password>@cluster0-eutgj.azure.mongodb.net/test?retryWrites=true
```

即可在环境变量中加入数据库地址

然后可在`node`中通过 `process.env.mongodb_url`访问。
这样可以在推上 github 时避免暴露用户名密码等敏感信息
