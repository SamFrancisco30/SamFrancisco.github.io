---
layout: post
title: LeetCode每日一题：图，深度搜索，并查集
categories: LeetCode Notes
description: LeetCode每日一题笔记：图，深度搜索，并查集
keywords: LeetCode Graph DFS
excerpt: LeetCode每日一题笔记
---

# 题目描述

![picture 0](../images/c3d846263802c282a84865e77b1fafb4fec7a152b0616f4f37c6784f9bc121d5.png)  


# Solution and Notes

## 1. 图的表示
题目给的无向图是一个边的二维数组，并不是一个直观的图的数据结构，在这里，我们使用邻接表来存储这个图

```
unordered_map<int, unordered_set<int>> graph;
```

之所以用这个形式而不是一个二维数组是因为这样可以节省空间