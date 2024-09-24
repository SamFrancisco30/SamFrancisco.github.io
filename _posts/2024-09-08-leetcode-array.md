---
layout: post
title: LeetCode数组类题目笔记
categories: LeetCode Notes
description: LeetCode数组类题目笔记
keywords: LeetCode Array
excerpt: LeetCode数组类题目笔记
---

# 双指针：对撞指针
基本的思路是设置头尾两个指针a和b，一个从数组的开头开始（左指针），另一个从数组的末尾开始（右指针），每次根据问题的条件移动其中一个或两个指针，直至满足某些特定条件。

基本格式：
```
int len = arr.length;
int a = 0;
int b = len-1;
while(a<b){
    if(满足题目的条件){
        // 处理答案
    }
    if(某个条件){
        a++;  // 移动左指针
    } else {
        b--;  // 移动右指针
    }
}
```
在以上示例代码中，每次移动完a或b都会进行一次条件判断，但有时我们可能需要连续移动a或者b指针，直到找到满足条件的元素，这时可以将if条件替换为while，使得指针可以跳过某些不符合条件的元素。例如，翻转字符串中的元音字母：
```
public String reverseVowels(String s) {
        int n = s.length();
        char[] arr = s.toCharArray();
        int i = 0, j = n - 1;
        while (i < j) {
            while (i < n && !isVowel(arr[i])) {
                ++i;
            }
            while (j > 0 && !isVowel(arr[j])) {
                --j;
            }
            if (i < j) {
                swap(arr, i, j);
                ++i;
                --j;
            }
        }
        return new String(arr);
    }
```
在此例中，while循环用于跳过不符合条件的字符（非元音字符），使得指针迅速移动到符合条件的位置，然后再执行交换操作。这种做法可以减少无效的判断次数，提高代码的效率。


# 双指针：滑动窗口
滑动窗口方法适用于需要求一个数组中满足特定条件的子数组的题目。

模板：
```
//外层循环扩展右边界，内层循环扩展左边界
for (int l = 0, r = 0 ; r < n ; r++) {
	//当前考虑的元素
	while (l <= r && check()) {//区间[left,right]不符合题意
        //扩展左边界
    }
    //区间[left,right]符合题意，统计相关信息
}
```

在外层循环中扩展右边界，不断扩大窗口，直到窗口不满足要求，此时再扩展左边界直到再次满足要求

例题：209，3，643，2555


# 单调栈
用于寻找数组中元素的下一个更大/更小的元素

模板（以单调递减栈为例）：
```
int[] result = new int[n];
Deque<Integer> stack = new ArrayDeque<>();

for (int i = 0; i < n; i++) {
    // 不断移除比新元素小的元素来保持升序
    while (!stack.isEmpty() && array[stack.peek()] < array[i]) {
        // 执行某种操作 (e.g., 根据栈顶元素做记录然后弹出栈顶元素)
        int index = stack.pop();
        result[index] = array[i];
    }
    // 添加新元素的下标到栈顶
    stack.push(i);
}
```

注意，栈里存放的是数组元素的下标而不是数组元素本身，这么做是便于更改result数组，因为：

每次从栈中弹出元素时，**新元素是比出栈元素向后找的更大的下一个元素**

如果是要找下一个更小的元素，就用递增栈，这样每次遇到更小元素就弹出栈顶的元素直到栈顶元素小于新元素

## 例题：柱状图中最大矩形
![](/images/posts/lc84.png)

思路：对于每一个bar，如果它是某个范围中的最低的bar，那么包括这个bar的最大矩形面积取决于这个范围能向右延申多少。而只要右边的bar比它高就能一直延申，所以可以使用单调递增栈来保存递增的bar。在延申的过程中，如果遇到比栈顶bar高度低的bar，说明这个范围不能再扩大了，我们就计算该范围里的最大面积。


代码：
```
class Solution {
    public int largestRectangleArea(int[] heights) {
        Deque<Integer> stack = new ArrayDeque<>();
        int maxArea = 0;
        // 必须要在最后加个0，来解决最后一个范围是递增的情况
        for (int i = 0; i <= heights.length; i++) {
            int h = (i == heights.length) ? 0 : heights[i];
            while (!stack.isEmpty() && h < heights[stack.peek()]) {
                int height = heights[stack.pop()];
                int width = stack.isEmpty() ? i : i - stack.peek() - 1;
                maxArea = Math.max(maxArea, height * width);
            }
            stack.push(i);
        }
        return maxArea;
    }
}
```





