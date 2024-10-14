---
layout: post
title: LeetCode题目笔记
categories: LeetCode Notes
description: LeetCode题目笔记
keywords: LeetCode
excerpt: LeetCode题目笔记
---

# 枚举
![](/images/posts/meiju.png)

对于这道题，常规暴力解法（两个for循环）复杂度是O(M*N)

使用哈希表枚举优化，思路：外循环遍历j，内循环再遍历i的话复杂度太高了，只需要遍历nums2[j]*k的倍数就行，然后看这个数有没有在nums1中（提前用哈希表存起来方便查找）

注意：
1. nums2[j]*k的倍数最大不能超过nums1数组中的最大数字，所以要提前记录下这个最大值
2. nums2也可以用哈希表存起来，因为可能会有重复的数字

```
class Solution {
    public long numberOfPairs(int[] nums1, int[] nums2, int k) {
        // 用map把两个数组的数字都存起来
        Map<Integer, Integer> count = new HashMap<>();
        Map<Integer, Integer> count2 = new HashMap<>();
        int max1 = 0;
        for (int num : nums1) {
            count.put(num, count.getOrDefault(num, 0) + 1);
            max1 = Math.max(max1, num);
        }
        for (int num : nums2) {
            count2.put(num, count2.getOrDefault(num, 0) + 1);
        }

        long res = 0;
        for (int a : count2.keySet()) {
            for (int b = a * k; b <= max1; b += a * k) {
                if (count.containsKey(b)) {
                    res += 1L * count.get(b) * count2.get(a);
                }
            }
        }
        return res;
    }
}
```

## 启示
1. 对于给定一个等式要求等式两边的值的题目，暴力二重循环复杂度太高，可以使用哈希表+枚举进行优化；
2. 枚举等式的一边得到一个值，然后看这个值是否valid（在本题中就是看nums1里是否存在）
3. 对于二重循环，要尽量减少外循环或内循环的循环次数，例如本题使用哈希表就可以使循环次数减少，重复的数字就只会遍历一次

# 172阶乘后的0
给定一个整数 `n` ，返回 `n!` 结果中尾随零的数量

思路：没法直接算出阶乘，因为很容易就会溢出。从数学角度进行分析，
乘数中只要有2*5就能得到一个10（结尾多一个0），因此只需要找乘数中2和5出现次数的最小值，但是5的次数一定是比2小的，所以
找5的因数个数就行

```
class Solution {
    public int trailingZeroes(int n) {
        int ans = 0;
        for(int i=1;i<=n;i++){
            int num = i;
            while(num>0){
                if(num%5==0){
                    ans++;
                    num/=5;
                }else{
                    break;
                }
            }
        }
        return ans;
    }
}
```

## 启示
1. 最直观的方法往往会超时或者溢出，要看一下题目给的范围