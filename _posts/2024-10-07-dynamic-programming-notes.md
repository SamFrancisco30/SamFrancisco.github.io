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
dp[i][j]:表示**从前i个物品里任意取，放进容量为j的背包的最大价值总和**

递归公式：
```
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
``` 

对于每一个物品i：
* 不放进背包：dp[i - 1][j]
* 放进背包：dp[i - 1][j - weight[i]] + value[i]

dp初始化：

dp[0][0]: 表示背包容量为0的情况，全部都是0，无需特殊处理

基本模板：
```
int[][] dp = new int[n+1][bagweight + 1];

// 第一件物品已经在初始化时装过了
// 外层遍历物品，从第二件开始
for (int i = 1; i <= n; i++) {
    // 内层遍历背包容量
    for (int j = 0; j <= bagweight; j++) {
        // 总容量不够装当前物体
        if (j < weight[i-1]) {
            dp[i][j] = dp[i - 1][j];
        }
        // 够装当前物体 
        else {
            dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weight[i-1]] + value[i-1]);
        }
    }
}

return dp[n][bagweight];
```

## 一维数组dp
```
int[] dp = new int[bagweight + 1];

// 第一件物品已经在初始化时装过了
// 外层遍历物品，从第二件开始
for (int i = 0; i < n; i++) {
    // 内层遍历背包容量
    for (int j = bagweight; j >= weight[i]; j--) {
        dp[j] = Math.max(dp[j], dp[j-weight[i]] + value[i]);
    }
}

return dp[bagweight];
```

要注意几点：
1. 这里的dp[j]的定义为容量为j的背包，所背的物品价值可以最大为dp[j]，这么定义就不需要额外初始化，
因为dp[0]为0
2. 有些题目中，dp[j]的定义会不同，一般会定义为从数组的前j个元素中选，使得满足某个条件，
此时dp[0]可能就是1
3. 对于0-1背包问题，一维数组dp的内循环遍历背包容量只能**从大到小**，这是为了保证物品i只被放入一次
4. 对于0-1背包问题，一维数组dp只能先遍历物品，不能先遍历背包容量

# 完全背包
完全背包与0-1背包的区别在于每个元素有无数个

## 一维数组dp
0-1背包的一维dp内循环遍历背包容量是从大到小遍历的，这是为了保证每个物品仅被添加一次。

然而对于完全背包，物品可以添加多次，所以要从小到大遍历背包容量
```
// 先遍历物品，再遍历背包
for(int i = 0; i < weight.size(); i++) { // 遍历物品
    for(int j = weight[i]; j <= bagWeight ; j++) { // 遍历背包容量
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);

    }
}
``` 
要注意：
1. 对于0-1背包问题，一维数组dp只能先遍历物品，不能先遍历背包容量.
但是对于完全背包来说，先遍历物品和背包都可以，取决于求的是组合还是排列



# 背包问题总结
## 一维dp数组定义
假设背包大小为n，dp一般这样定义：
```
int[] dp = new int[n+1];
```
dp[j]的含义可能是:
* 前j个元素装进背包的最大价值
* 前j个元素装满背包的方法

## 初始化
根据dp数组的定义不同，初始化的方法也不同。

dp[j]的含义可能是:
* 前j个元素装进背包的最大价值：dp[0] = 0;
* 前j个元素装满背包的方法：dp[0] = 1;

## 递推公式
如果dp[j]的含义是前j个元素装满背包的方法，那么递推公式就是：
```
dp[j] += dp[j - nums[i]];
```

## 内外循环顺序（仅针对完全背包）
1. 如果求组合数就是外层for循环遍历物品，内层for遍历背包
2. 如果求排列数就是外层for遍历背包，内层for循环遍历物品

求排列数的意思是[1,2]和[2,1]是两个不同的答案


# 走迷宫类型
从迷宫左上角向右下角走，只能向下或向右。可以只用一个一维数组来dp，
这是因为dp[i+1][j+1] 的状态转移，仅为其上一行状态 dp[i][j+1]和前一位状态 dp[i+1][j]有关