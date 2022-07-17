---
title: Q&A
date: 2022-07-17 17:53:37
---

> 记录一下日常开发中遇到的问题

<!-- more -->

###### VSCode 安装 Go 第三方报 timeout 错误？

在设置里面配置 HTTP 代理，代理地址在 ShadowsocksX-NG 偏好设置 HTTP 标签页查看
*Mac*

###### VSCode 扩展 Vim 无法长按键盘键？

终端执行 `defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false`，重启 VSCode
*Mac*

###### Go全局包执行报 command not found 错误？

将全局包安装路径赋值给 PATH 变量，如 `export PATH=$PATH:/Users/everchan/go/bin`
*Mac*

###### zsh 配置文件？

在用户目录下面的 .zshrc 文件，没有则创建

###### 编辑 PAC 用户自定义规则？

格式: `!!<domain>`

#### bash 短横线 - 含义？

表示标准输出，或从标准输出中获得输入
