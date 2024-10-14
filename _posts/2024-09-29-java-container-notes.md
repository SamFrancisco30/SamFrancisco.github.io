---
layout: post
title: Java集合笔记
categories: Java
description: Java集合笔记
keywords: Java
excerpt: Java Container Notebook
---

# Java集合框架
Java 集合，也叫作容器，主要是由两大接口派生而来：一个是 Collection接口，主要用于存放单一元素；另一个是 Map 接口，主要用于存放键值对。对于Collection 接口，下面又有三个主要的子接口：List、Set 、 Queue

![](/images/posts/java_container.png)

# List
## ArrayList
ArrayList 的底层是数组队列，相当于动态数组。底层是**Object[]**，是**线程不安全**的

方法：
```
// 初始化一个 String 类型的 ArrayList
ArrayList<String> stringList = new ArrayList<>(Arrays.asList("hello", "world", "!"));

// 添加元素到 ArrayList 中
stringList.add("goodbye");

// 访问元素
String s = stringList.get(0);

// 修改 ArrayList 中的元素
stringList.set(0, "hi");

// 删除 ArrayList 中的元素
stringList.remove(0);

// 排序
Collections.sort(sites);

// 转化为数组
String[] arr = stringList.toArray();
```

### 底层原理
使用无参构造函数初始化 ArrayList 后，它当时的数组容量为 0。当后面调用add()进行添加操作时，将会给数组分配默认的初始容量为DEFAULT_CAPACITY = 10

扩容：ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）

## LinkedList 
底层使用双向链表，同样线程不安全

可以向头尾添加/移除/获取元素：
```
LinkedList<String> sites = new LinkedList<String>();

sites.addFirst("Google");

sites.addLast("Google");

sites.removeFirst();

sites.removeLast();

String s = sites.getFirst();

String s = sites.getLast()
```

## 二者区别
1. LinkedList不支持高效的随机元素访问，而 ArrayList（实现了 RandomAccess 接口） 支持
2. ArrayList的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）
3. LinkedList在头尾添加元素的复杂度是O(1)

# Set
## HashSet
底层用HashMap来实现，值是一个constant dummy value(new Object() )

特性：
* 元素无序
* 无重复元素
* 线程不安全

可使用`Collections.synchronizedSet`来确保线程安全

方法：
```
HashSet<String> sites = new HashSet<String>();

sites.add("Google);

// 判断元素是否存在
if(sites.contains("Baidu")){
    // ...
}

// 删除元素，删除成功返回 true，否则为 false
sites.remove("Taobao");

// 删除所有元素
sites.clear();

// 迭代HashSet
for (String i : sites) {
    System.out.println(i);
}
```

## LinkedHashSet
LinkedHashSet是Hashset的子类，不同点在于：
1. 保持元素的插入和取出顺序（FIFO）
2. 底层使用**LinkedHashMap**(table数组+doubly linked list)

## TreeSet
TreeSet的底层是TreeMap（红黑树实现）

元素是有序的，排序的方式有自然排序和定制排序

插入/删除/查询的时间复杂度是O(log(n))

## 总结
HashSet、LinkedHashSet 和 TreeSet 都是 Set 接口的实现类，都能保证元素唯一，并且都不是线程安全的。

HashSet、LinkedHashSet 和 TreeSet 的主要区别在于底层数据结构不同。HashSet 的底层数据结构是哈希表（基于 HashMap 实现）。LinkedHashSet 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。TreeSet 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。

底层数据结构不同又导致这三者的应用场景不同。HashSet 用于不需要保证元素插入和取出顺序的场景，LinkedHashSet 用于保证元素的插入和取出顺序满足 FIFO 的场景，TreeSet 用于支持对元素自定义排序规则的场景。


# Queue
Queue是单端队列，只能从一端进一端出，遵循FIFO原则

Queue提供两类方法：一种在操作失败后会抛出异常，另一种则会返回特殊值

抛出异常：
```
Queue<TreeNode> queue = new LinkedList<>();

queue.add(root);

queue.remove();

queue.element();
```

返回特殊值：
```
queue.offer(root);

queue.poll();

queue.peek();
```

检查元素是否存在：
```
boolean flag = queue.contains(root);
```

## ArrayDeque与LinkedList区别
1. ArrayDeque是基于**resizable array和双指针(头尾)**来实现，而LinkedList则通过链表来实现

2. ArrayDeque**不支持存储 NULL 数据**，但LinkedList支持。

3. ArrayDeque插入时可能存在扩容过程, 不过均摊后的插入操作依然为 O(1)。虽然LinkedList不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢


## PriorityQueue
```
PriorityQueue<Integer> minHeap = new PriorityQueue<Integer>(); //minheap，默认容量为11
PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>(11,new Comparator<Integer>(){ //maxheap，容量11
    @Override
    public int compare(Integer i1,Integer i2){
        return i2-i1;
    }
});

// 或者
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);


PriorityQueue<Map.Entry<Integer, Integer>> minHeap = new PriorityQueue<>(
    (a, b) -> a.getValue() - b.getValue()
);

PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```

对于k-largest问题，用minheap，堆顶元素是第k大的；

对于k-smallest问题，用maxheap，堆顶元素是第k小的。
