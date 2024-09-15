---
layout: post
title: LeetCode链表类题目笔记
categories: LeetCode Notes
description: LeetCode链表类题目笔记
keywords: LeetCode LinkedList
excerpt: LeetCode链表类题目笔记
---

# 环形链表（检查是否有环）
## 哈希表
用一个哈希表来存放访问过的节点
```
Set<ListNode> seen = new HashSet<ListNode>();
```
时间复杂度：O(N)

空间复杂度：O(N)

## 快慢指针
快指针一次走两步，慢指针一次走一步，有环的话一定最终会相遇
```
public boolean hasCycle(ListNode head) {
    if (head == null)
        return false;
    //快慢两个指针
    ListNode slow = head;
    ListNode fast = head;
    while (fast != null && fast.next != null) {
        //慢指针每次走一步
        slow = slow.next;
        //快指针每次走两步
        fast = fast.next.next;
        //如果相遇，说明有环，直接返回true
        if (slow == fast)
            return true;
    }
    //否则就是没环
    return false;
}
```
时间复杂度：O(N)

空间复杂度：O(1)

# 寻找环形链表入环处
同样可以使用哈希表

若采用快慢指针法，则在两指针相遇后，**将快指针移动到链表头，随后两指针一起每次移动一步**，最终会在入环处相遇
```
    public ListNode detectCycle(ListNode head) {
         ListNode slow = head, fast = head;
        //快慢指针相遇
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            //第一次相遇退出循环
            if (slow == fast) break;
        }
        //判断是否有环 
        if(fast==null||fast.next==null)return null;
        //有环则将fast移动至head并移动S2距离
        fast=head;
        while(fast!=slow){
            slow=slow.next;
            fast=fast.next;
        }
       return fast;
    }
```

# 相交链表
## 哈希表
先遍历一个链表，将所有节点加入哈希表中，再遍历另一个链表看节点是否已经存在

## 双指针
1. 当链表 headA 和 headB 都不为空时，创建两个指针 pA 和 pB，初始时分别指向两个链表的头节点 headA 和 headB。

2. while循环检查两个指针是否相等

3. 在while循环中每次都移动两个指针：若pA为空则移动到headB；若pB为空则移动到headA

```
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) {
        return null;
    }
    ListNode pA = headA, pB = headB;
    while (pA != pB) {
        pA = pA == null ? headB : pA.next;
        pB = pB == null ? headA : pB.next;
    }
    return pA;
}
```

# 删除链表倒数第N个节点
## 使用栈
将元素全部放入栈中，再弹出N个元素，此时栈顶元素就是倒数第N个元素的前驱元素
```
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        Deque<ListNode> stack = new LinkedList<ListNode>();
        ListNode cur = dummy;
        while (cur != null) {
            stack.push(cur);
            cur = cur.next;
        }
        for (int i = 0; i < n; ++i) {
            stack.pop();
        }
        ListNode prev = stack.peek();
        prev.next = prev.next.next;
        ListNode ans = dummy.next;
        return ans;
    }
}
```

缺点：空间复杂度为O(L)，其中 L 是链表的长度

## 双指针
可以使用两个指针 first 和 second 同时对链表进行遍历，并且 first 比 second 超前 n 个节点。当 first 遍历到链表的末尾时，second 就恰好处于倒数第 n 个节点

```
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        ListNode first = head;
        ListNode second = dummy;
        for (int i = 0; i < n; ++i) {
            first = first.next;
        }
        while (first != null) {
            first = first.next;
            second = second.next;
        }
        second.next = second.next.next;
        ListNode ans = dummy.next;
        return ans;
    }
}
```

## Dummy Head
上述两个例子中都创建了一个dummy head，这是因为对于单链表，要删除/增加一个节点需要知道其前驱节点，但是头节点不存在前驱节点，所以可以加一个dummy head来作为头节点的前驱节点

# 反转链表
## 迭代式写法
每遍历到一个节点就将其next指向前一个节点
```
public ListNode reverseList(ListNode head) {
    ListNode cur = head;
    ListNode pre = null;
    while(cur!=null){
        ListNode tmp = cur.next;
        cur.next = pre;
        pre = cur;
        cur = tmp;
    }
    return pre;
}
```

## 递归式写法
```
public ListNode reverseList(ListNode head) {
    // Base case
    if(head==null || head.next==null){
        return head;
    }

    // 假设从head.next开始的后续链表已经反转完成，获取到新的头节点
    ListNode newHead = reverseList(head.next);

    // 剩下的head需要加入后面的反转好了的链表中
    head.next.next = head;
    head.next = null;

    // 返回新的头节点
    return newHead;
}
```