---
title: 如何编写 Makefile 文件
date: 2022-07-17 21:07:55
categories: 后端
---

> 如何编写 Makefile 文件

<!-- more -->

# 格式

```makefile
<target> : <prerequisites> 
[tab]  <commands>
```

- target: 目标
- prerequisites: 前置条件
- commands: 命令

```bash
make <target>
```

# 参考

- [Make 命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)
