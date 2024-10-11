---
layout: post
title: LeetCode动态规划题目笔记
categories: LeetCode Notes
description: LeetCode动态规划题目笔记
keywords: LeetCode DP
excerpt: LeetCode动态规划题目笔记
---
# 0-1背包
问题：有n件物品和一个最多能背重量为w 的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。每件物品只能用一次，求解将哪些物品装入背包里物品价值总和最大

## 二维数组dp
dp[i][j]:表示**从下标为[0-i]的物品里任意取，放进容量为j的背包的最大价值总和**

递归公式：
```
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
``` 

对于每一个物品i：
* 不放进背包：dp[i - 1][j]
* 放进背包：dp[i - 1][j - weight[i]] + value[i]

dp初始化：

dp[i][0]: 表示背包容量为0的情况，全部都是0，无需特殊处理

dp[0][j]: i为0，存放编号0的物品的时候，各个容量的背包所能存放的最大价值。

基本模板：
```
int[][] dp = new int[n][bagweight + 1];


for (int j = weight[0]; j <= bagweight; j++) {
    dp[0][j] = value[0];
}

// 第一件物品已经在初始化时装过了
// 外层遍历物品，从第二件开始
for (int i = 1; i < n; i++) {
    // 内层遍历背包容量
    for (int j = 0; j <= bagweight; j++) {
        // 总容量不够装当前物体
        if (j < weight[i]) {
            dp[i][j] = dp[i - 1][j];
        }
        // 够装当前物体 
        else {
            dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
        }
    }
}
```

# 走迷宫类型
从迷宫左上角向右下角走，只能向下或向右。可以只用一个一维数组来dp，
这是因为dp[i+1][j+1] 的状态转移，仅为其上一行状态 dp[i][j+1]和前一位状态 dp[i+1][j]有关