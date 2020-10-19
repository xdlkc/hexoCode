---
title: 解决mac系统vscode code helper占用CPU过高问题
date: 2020-10-19 00:09:59
tags:
---

（转载自网络答案）

<!--more-->

在vscode中添加以下配置：

```json
"files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.hg": true,
    "**/CVS": true,
    "**/.DS_Store": true,
    "**/tmp": true,
    "**/node_modules": true,
    "**/bower_components": true,
    "**/dist": true
},
"files.watcherExclude": {
    "**/.git/objects/**": true,
    "**/.git/subtree-cache/**": true,
    "**/node_modules/**": true,
    "**/tmp/**": true,
    "**/bower_components/**": true,
    "**/dist/**": true
}
```

