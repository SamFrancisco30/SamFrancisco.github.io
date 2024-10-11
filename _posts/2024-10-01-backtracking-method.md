---
layout: post
title: Backtracking 回溯法
categories: Programming
description: 回溯法复习及思考
keywords: Backtracking, Programming
excerpt: 此部分介绍个人对回溯法的学习和总结
---
# 回溯法的本质
**在集合中递归查找子集**

回溯法可以抽象为树形结构，集合的大小是树的宽度；递归的深度是树的深度

# 回溯法可以解决哪些问题：
1. 组合，N个里面选K个
2. 切割，比较类似组合
3. 子集，一个集合里求有多少符合条件的子集
4. 排列
5. 棋盘
   
# 递归法模板
```
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}
```
for循环：在当前层次的当前集合中横向遍历
backtracking：纵向遍历，树的深度加深

# 组合问题
## N个元素中选K个
图示：
![](/images/2023-03-30-22-00-55.png)
```
List<List<Integer>> result = new ArrayList<>();  // Store the results
Deque<Integer> path = new ArrayDeque<>();          // Store the current path

public List<List<Integer>> combine(int n, int k) {
    backtracking(n, k, 1);
    System.out.println(result);
    return result;
}

public void backtracking(int n, int k, int start) {
    if (path.size() == k) {                      // If the path has reached the desired length
        result.add(new ArrayList<>(path));       // Add a copy of the path to the results
        return;
    }
    for (int i = start; i <= n; i++) {
        path.add(i);                             // Process the current node
        backtracking(n, k, i + 1);               // Recursive call
        path.removeLast();            // Backtrack and remove the last element
    }
}
```

## 什么时候需要start
如果是在同一个集合来求组合的话，就需要start

如果是多个集合取组合，各个集合之间相互不影响，那么就不用start，如LC 17

## 剪枝优化
为什么要优化：有时候for循环中从start开始处之后的元素个数已经不足了，即使全部选完也不够K个

需要优化的部分：
```
for (int i = start; i <= n; i++)
```

已经选择的元素个数：`path.size()`
还需要的元素个数：`k - path,size()`

优化之后的for循环：
```
for (int i = start; i <= n - (k - path.size()) + 1; i++) 
```