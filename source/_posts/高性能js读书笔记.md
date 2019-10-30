---
title: 高性能js读书笔记
date: 2018-08-07 12:47:56
tags:
---

### 动态脚本元素

```js 
var script = document.createElement("script");
script.type = "text/javascript";
script.src = "file1.js";
document.getElementsByTagName("head")[0].appendChild(script);
```

这种技术的重点在于：无论何时启动下载，文件的下载和执行过程都不会阻塞页面其他进程。