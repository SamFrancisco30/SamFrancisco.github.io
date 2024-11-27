---
layout: post
title: LeetCode贪心题目笔记
categories: LeetCode Notes
description: LeetCode贪心题目笔记
keywords: LeetCode Greedy
excerpt: LeetCode贪心题目笔记
---

# 无重叠区间
题目：你今天有好几个活动，每个活动都可以用区间 [start,end] 表示开始和结束的时间，请问你今天最多能参加几个活动呢？

贪心思路：**优先选择参加那些结束时间早的**，因为这样可以留下更多的时间参加其余的活动