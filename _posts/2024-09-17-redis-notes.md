---
layout: post
title: Redis学习笔记
categories: Redis
description: Redis学习笔记
keywords: Redis
excerpt: Redis Notebook
---

# 基本命令
* SET
* GET
* DELETE
* TTL：查看键的过期时间
* EXPIRE [key] [second]：设置键的过期时间
* SETEX [key] [second] [value] ：设置带过期时间的键值对
* KEYS [pattern]：查看有哪些键

    e.g. KEYS * 查看所有键

