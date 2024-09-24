---
layout: post
title: 零碎的Java笔记
categories: Java
description: 阅读项目源码中零碎的Java笔记
keywords: Java
excerpt: Java Notebook
---
# 数组相关
## 数组的遍历
1. 传统的for循环+下标
2. for-each：只能遍历但**不能修改元素**
```
int[] arr = {1, 2, 3, 4, 5};
        
        // This will not modify the array
        for (int num : arr) {
            num = num * 2;  // Does not modify the original array
        }
```

## 修改所有元素：`Arrays.setAll()`或者 `Arrays.fill()`
```
int[] arr = {1, 2, 3, 4, 5};
        
// Modify array elements using Arrays.setAll
Arrays.setAll(arr, i -> arr[i] * 2);

// Or fill it with a constant value
Arrays.fill(arr, -1);
```

## 数组排序：`Arrays.sort()`
默认升序：
```
int[] arr = {5, 2, 9, 1, 5, 6};
        
// Sort the array
Arrays.sort(arr);
```

降序：
```
int[] arr = {5, 2, 9, 1, 5, 6};
        
// Sort the array in descending order
Arrays.sort(arr, Collections.reverseOrder());
```

# 双端队列deque
## deque初始化
```
Deque<Integer> deque = new ArrayDeque<>();
```

## 插入元素
```
deque.addFirst(element);
deque.addLast(element);
```

## 移除元素
```
deque.pollFirst(); // or deque.removeFirst();
deque.pollLast(); // or deque.removeLast();
```

## 访问头尾元素
```
deque.peekFirst();
deque.peekLast();
```

## 用途
在一些题目中，我们需要记录一个滑动窗口里的数值的最大/最小值，此时可以利用deque构造一个单调队列

例如，如果想记录最大值，就构造一个单调递减队列，使得队头的元素最大

### maximum deque
1. 添加元素前，**从队尾**不断移除比新元素小的元素
2. 添加新元素**到队尾**
3. 最大的元素在**队头**

e.g.
```
while (!deque.isEmpty() && deque.peekLast() <= newElement) {
    deque.pollLast();
}
deque.addLast(newElement);

// 队头元素即为当前窗口的最大值
int maxValue = deque.peekFirst();
```

*注意：当移动窗口的左边界时，需要检查队头的元素是否已经不在窗口内，并在必要时从队头移除。*

### stack
也可以将Deque用作栈，使用传统的push/pop/peek方法

# 字符串相关
Java中字符串相关类型有3种：String，StringBuilder和StringBuffer。其中，String不可变，其余两种是可变字符串，但是StringBuilder线程不安全
## String
String类存储字符串的方式是通过char型数组存储：
```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
}
```
## String常量和String对象
JVM针对String类型有一个常量池（堆区），如果是字符串常量形式的声明首先会查看常量池中是否存在这个常量，如果存在就不会创建新的对象，否则在常量池中创建该字符串并创建引用。此后无论式创建多少个相同的字符串都是指向这一个地址的引用，而不再开辟新的地址空间。

但对于字符串对象每次new都会在堆区形成一个新的内存区域并填充相应的字符串，不论堆区是否已经存在该字符串。

从以下例子可以看出字符串常量不可修改，只是引用的位置更改了
```
String s1 = "abc";
s1 += "d";
System.out.println(s1); // "abcd" 
// 内存中有"abc"，"abcd"两个对象，s1从指向"abc"，改变指向，指向了"abcd"。
```

## 字符串的比较
```
s1.equals(s2);             // 比较内容是否相等
s1.equalsIgnoreCase(s2)    // 比较内容是否相等，忽略大小写

System.out.println(s1 == s2); // 比较字符串对象地址是否相等
```

## 其他重要方法
public int length () ：返回此字符串的长度。

public String concat (String str) ：将指定的字符串连接到该字符串的末尾。

public char charAt (int index) ：返回指定索引处的 char值。

public int indexOf (String str) ：返回指定子字符串第一次出现在该字符串内的索引。

public char[] toCharArray () ：将此字符串转换为新的字符数组。

public String substring (int beginIndex, int endIndex) ：返回一个子字符串，从beginIndex到endIndex截取字符串。含beginIndex，不含endIndex。

## StringBuilder(a mutable sequence of characters)
```
StringBuilder sb = new StringBuilder("Hello");

sb.append(" World"); // 添加一个字符串或字符

sb.insert(5, " World"); // 向某个index处插入字符串

sb.delete(5, 11); // 删除一个子字符串

sb.deleteCharAt(5);

sb.reverse();

String result = sb.toString(); // 转换为String
```



# Finally
在Java中，finally 关键字用于创建一个代码块，它跟在一个try块之后，可选地跟在一个或多个catch块之后。无论try块内发生什么情况（是否发生异常），finally 块中的代码几乎总是会被执行。finally块通常用于清理资源，比如关闭文件、释放内存、释放锁等，以确保这些代码无论正常逻辑还是异常处理都能得到执行。

示例代码：
```
public RaftProto.VoteResponse requestVote(RaftProto.VoteRequest request) {
        raftNode.getLock().lock();
        try {
            RaftProto.VoteResponse.Builder responseBuilder = RaftProto.VoteResponse.newBuilder();
            responseBuilder.setGranted(false);
            responseBuilder.setTerm(raftNode.getCurrentTerm());
            if (!ConfigurationUtils.containsServer(raftNode.getConfiguration(), request.getServerId())) {
                return responseBuilder.build();
            }
            if (request.getTerm() < raftNode.getCurrentTerm()) {
                return responseBuilder.build();
            }
            if (request.getTerm() > raftNode.getCurrentTerm()) {
                raftNode.stepDown(request.getTerm());
            }
            boolean logIsOk = request.getLastLogTerm() > raftNode.getLastLogTerm()
                    || (request.getLastLogTerm() == raftNode.getLastLogTerm()
                    && request.getLastLogIndex() >= raftNode.getRaftLog().getLastLogIndex());
            if (raftNode.getVotedFor() == 0 && logIsOk) {
                raftNode.stepDown(request.getTerm());
                raftNode.setVotedFor(request.getServerId());
                raftNode.getRaftLog().updateMetaData(raftNode.getCurrentTerm(), raftNode.getVotedFor(), null, null);
                responseBuilder.setGranted(true);
                responseBuilder.setTerm(raftNode.getCurrentTerm());
            }
            LOG.info("RequestVote request from server {} " +
                            "in term {} (my term is {}), granted={}",
                    request.getServerId(), request.getTerm(),
                    raftNode.getCurrentTerm(), responseBuilder.getGranted());
            return responseBuilder.build();
        } finally {
            raftNode.getLock().unlock();
        }
    }
```

需要注意的是：

1. 如果try块中包含return语句，finally块仍然会在返回之前执行。
2. 如果在finally块中有return语句，它将覆盖try或catch块中的return值。因此，最好避免在finally块中使用return。

*finally块不一定会执行的情况包括：在try或catch块中调用System.exit()等终止JVM的方法，或者当前线程被杀死*

# ConcurrentMap
HashMap在并发环境下使用中最为典型的一个问题，就是在HashMap进行扩容重哈希时导致*Entry链形成环*。一旦Entry链中有环，势必会导致在同一个桶中进行插入、查询、删除等操作时陷入*死循环*。

ConcurrentMap是HashMap的一个线程安全的、支持高效并发的版本。在默认理想状态下，ConcurrentHashMap可以支持16个线程执行并发写操作及任意数量线程的读操作。

# ReentrantLock
Java中的ReentrantLock类属于*可重入锁独占锁*，它提供了与synchronized关键字类似的功能，但具有更高的灵活性和可配置性。

特点:
* 可重入：一个线程可以多次获取同一个锁。
* 中断响应：支持中断，可以响应中断。
* 公平性：可以选择公平或非公平锁。

默认情况下，ReentrantLock是非公平的，也就是说，不能保证先请求锁的线程先获得锁。公平锁则会按照请求锁的顺序分配锁。可以通过在构造ReentrantLock时传递true来实现公平锁

# 线程池相关：ExecutorService
ExecutorService 是 Java 中一个用于管理线程池和并发任务执行的框架。它是 java.util.concurrent 包的一部分。

ExecutorService 通过使用线程池来抽象线程的创建、管理和操控，使得concurrency更加简单高效

## ExecutorService如何管理线程池
当我们submit一个task时，ExecutorService会检查线程池中有无空闲线程，如果有则分配；若无则将task放到一个queue中等待有线程available。

## 线程池类型

1. FixedThreadPool: : A thread pool with a fixed number of threads.

```
ExecutorService executorService = Executors.newFixedThreadPool(5);
```

2. CachedThreadPool: A thread pool that creates new threads as needed but reuses previously constructed ones when available.

```
ExecutorService executorService = Executors.newCachedThreadPool();
```

CachedThreadPool能够根据需要创建新线程，并在空闲时重用已有的线程，它的线程数量不固定，可以根据任务的需求动态调整

通常使用CachedThreadPool是在前端调用有限制的情况， 比如在tomcat 中，tomcat 本身有线程池数量限制，那么它在代码内使用共享 的CachedThreadPool 是有调用数量的保证的。比如知道需要并发运行的次数，用cachedThreadPool 是可以的。


3. SingleThreadExecutor: 单线程池，只有一个线程工作，确保所有任务按照指定顺序（FIFO, LIFO, 优先级）执行。

```
ExecutorService executorService = Executors.newSingleThreadExecutor();
```

比起直接使用单一线程，SingleThreadExecutor的好处有：
* 任务执行完毕后线程不会立即销毁，而是保留在线程池中等待下一个任务。这样可以*避免频繁地创建和销毁线程带来的开销*
* 保证任务按照提交的顺序依次执行，不会产生并发访问的问题

4. ScheduledThreadPool: 定时线程池，用于执行定时任务或周期性任务，可以指定任务的延迟时间或执行周期。

```
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
```