---
title: rsync-deploy
date: 2019-03-07 09:18:43
tags:
---

```
command rsync
args [ '--delete',
'-v',
'-az',
'-e',
'ssh -p 22',
'/Users/liuqi/github/blog/public/',
'ubuntu@132.232.17.140:/var/www/www.abouttime.cn' ]
options { verbose: true }
```
