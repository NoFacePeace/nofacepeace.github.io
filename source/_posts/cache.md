---
title: 缓存那些事
date: 2022-07-21 21:25:19
categories: 后端
---

> 记录一些缓存相关的东西

<!-- more -->

## 本地缓存 vs. 分布式缓存

https://dzone.com/articles/process-caching-vs-distributed

## 缓存问题

### 本地缓存重启穿透

1. 缓存预热
2. singleflight

### 缓存穿透

1. 查询不存在的缓存
2. 缓存空结果
3. 布隆过滤器

### 缓存击穿

1. key 刚好过期
2. 互斥锁
3. 不过期

### 缓存雪崩

1. 多个 key 同时过期
2. 互斥锁
3. 过期时间随机


