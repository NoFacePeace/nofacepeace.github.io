---
title: 搭建个人博客
date: 2018-01-29 05:36:40
tags:
- Hexo
- Github
categories:
- 教程 
---

学习用 Hexo + Github Pages 搭建个人博客,并通过自己的域名访问博客.

<!-- more -->

# 搭建环境

## 安装 Git

## 安装 nvm 和 Node.js

## 安装 Hexo

```bash
# -g 全局安装
npm install -g hexo-cli
```

# 初始化

```bash
# 没有指定文件夹将会在当前文件夹新建所需要的文件
hexo init <folder>
cd <folder>
npm install
```

# 常用的命令

```bash
# layout
#  - post default
#  - page
#  - draft
hexo new [layout] <title>
# 清除缓存文件和已生成的静态文件
hexo clean
# 生成静态文件
hexo g
# 部署网站
hexo d
# 启动服务器
hexo server
```

# 个性化设置

## 站点配置文件

```yml
# _config.yml
# 注意冒号后空格
title: 网站标题
subtitle: 网站副标题
description: 网站描述
author: 名字
language: 语言
url: 网址
theme: next
```

## 主题配置文件

### 下载主题

```bash
git clone git@github.com:iissnan/hexo-theme-next.git themes/next
```

### 修改配置

```yml
# ./themes/next/_config.yml
# 修改外观
scheme: Gemini
# 设置菜单
menu:
  home: / || home
  about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
# 设置头像
# source/images/logo.jpg
avatar: /images/logo.jpg
# 设置背景动画
canvas_nest: true
# 设置侧边栏
display: always
```

### 添加页面

```bash
# 添加标签页面
hexo new page tags
# 添加分类页面
hexo new page categories
# 添加关于页面
hexo new page about
```

### 设置页面类型

```markdown
# source/tags/index.md
title: 标签
type: "tags"
# source/categories/index.md
title: 分类
type: "categories"
# source/about/index.md
title: 关于我
type: "about"
```

# 部署Git

## Github创建仓库

```md
# 格式必需
Repository name: <accountName>.github.io
# 描述非必需
Description:
# 权限必需
public
# initialize非必需
initialize this repository with a README
```

## 安装 hexo-deploer-git

```bash
npm install hexo-deploer-git --save
```

## 配置

```yml
# _config.yml 站点配置文件
deploy:
  type: git
  repo: <repository url>
  branch: [branch]
```

## 添加 README.md

```bash
# ./public/
touch README.md
vim README.md
```

## 部署命令

```bash
hexo g -d
```

# 域名

## 域名解析

```md
主机记录: 域名
记录类型: CNAME
记录值: <accountName>.github.io
```

## 域名映射

```bash
# ./source
echo blog.orzcn.cn > CNAME
hexo g -d
```

# 写作

## 新建文章

```bash
hexo new <title>
```

## Front-matter

```md
# Front-matter 是文件最上方以 --- 分隔的区域,用来指定个别文件的变量
title: 标题
date: 建立日期
updated: 更新日期
tags: 标签
categories: 分类
```

## 摘要

```md
# 上方的文字作为文章的摘要
<!-- more -->
```

# 扩展

## 添加 Local Search

1. 安装 hexo-generator-searchdb，在站点的根目录下执行以下命令

  ```bash
  npm install hexo-generator-searchdb --save
  ```

1. 编辑站点配置文件，新增以下内容到任意位置

  ```config
  search:
    path: search.xml
    field: post
    format: html
    limit: 10000
  ```

1. 编辑主题配置文件，启用本地搜索功能

  ```config
  local_search:
    enable: true
  ```