---
layout: post
title: LeetCode二叉树题目笔记
categories: LeetCode Notes
description: LeetCode二叉树题目笔记
keywords: LeetCode Array
excerpt: LeetCode二叉树题目笔记
---
# 前中后序遍历
## 递归法
无论是哪种遍历，递归函数的base case都一样，只是后面三条语句的顺序不同

模板代码：
```
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        recursive(root, result);
        return result;
    }
    public void recursive(TreeNode p, List<Integer> result){
        if(p==null){
            return;
        }
        result.add(p.val);
        recursive(p.left, result);
        recursive(p.right, result);
    }
}
```

## 迭代法
迭代法使用栈来辅助操作，对于前序和后序遍历，我们可以先将根节点入栈，随后while循环判断栈是否为空，在while循环的一开始将栈顶元素出栈，随后尝试将出栈的元素的右左子节点先后入栈。

对于前序遍历，**先入右节点，再入左节点**，这样就可以先出左节点再出右节点

前序遍历模板代码：
```
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        //加个边界条件判断
        if (root == null)
            return res;
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);//压栈
        while (!stack.empty()) {
            TreeNode cur = stack.pop();//出栈
            res.add(cur.val);
            if (cur.right != null) {
                stack.push(cur.right);
            }
            if (cur.left != null) {
                stack.push(cur.left);
            }
        }
        return res;
    }
}
```
对于后序遍历，只需要稍微改变一下前序遍历的代码，**先入左节点，再入右节点，最后将结果反转即可**

后序遍历模板代码：
```
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        //加个边界条件判断
        if (root == null)
            return res;
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);//压栈
        while (!stack.empty()) {
            TreeNode cur = stack.pop();//出栈
            res.add(cur.val);
            if (cur.left != null) {
                stack.push(cur.left);
            }
            if (cur.right != null) {
                stack.push(cur.right);
            }
        }
        Collections.reverse(res);
        return res;
    } 
}
```
中序遍历的代码稍有不同，不需要一开始就将根节点入队，也不需要在while循环的一开始就将栈顶元素出队，因为我们需要一直向左边遍历：

```
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        //加个边界条件判断
        if (root == null)
            return res;
        Stack<TreeNode> stack = new Stack<>();
        while (root != null || !stack.isEmpty()) {
            while (root != null) {
                stack.push(root);
                root = root.left;
            }
            if (!stack.isEmpty()) {
                root = stack.pop();
                res.add(root.val);
                root = root.right;
            }
        }
        return res;
    }
}
```

**对于二叉搜索树，它的中序遍历的结果一定是有序的**

# 层序遍历
思路：使用队列储存每一层的节点进行BFS，每次while循环里queue中的元素就是某一层的元素，依次将它们出队、记录、并将其子节点入队。

模板代码：
```
public List<List<Integer>> levelOrder(TreeNode root) {
    //边界条件判断
    if (root == null)
        return new ArrayList<>();

    //队列
    Queue<TreeNode> queue = new LinkedList<>();
    List<List<Integer>> res = new ArrayList<>();

    //根节点入队
    queue.add(root);

    //如果队列不为空就继续循环
    while (!queue.isEmpty()) {

        //BFS打印，levelNum表示的是每层的结点数
        int levelNum = queue.size();

        //subList存储的是每层的结点值
        List<Integer> subList = new ArrayList<>();
        for (int i = 0; i < levelNum; i++) {
            //出队
            TreeNode node = queue.poll();
            subList.add(node.val);

            //左右子节点如果不为空就加入到队列中
            if (node.left != null)
                queue.add(node.left);
            if (node.right != null)
                queue.add(node.right);
        }
        //把每层的结点值存储在res中，
        res.add(subList);
    }
    return res;
}
```