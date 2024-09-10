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