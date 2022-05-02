---
title: 获取 ETH
date: 2022-05-02 10:33:10
tags:
  - 区块链
categories: 教程
---

> 获取 ETH 教程

<!-- more -->

## 环境

- 操作系统 CentOS 8
- 安装 CUDA
  https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=CentOS&target_version=8&target_type=rpm_local

## 矿池 F2Pool

- 注册账户
- 添加挖矿账号
- 添加钱包地址

## 代理服务器

- 安装 Realm
  https://github.com/zhboner/realm
- 启动 Realm

```bash
nohup ./realm -l 0.0.0.0:16688 -r eth.f2pool.com:6688 > tmp 2>&1 &
```

## 工具 Bminer

- 下载
  https://www.bminer.me/
- 上传服务器

```
rz bminer-v16.4.11-2849b5c-amd64.tar.xz
```

- 解压

```bash
tar -xvJf bminer-v16.4.11-2849b5c-amd64.tar.xz
```

- 启动

```bash
nohup ./bminer -uri ethstratum://<F2Pool 账号>.<矿工名>@<proxy IP>:port > temp 2>&1 &
```
