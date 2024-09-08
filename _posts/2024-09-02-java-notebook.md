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

## 修改所有元素：`Arrays.setAll()`
```
int[] arr = {1, 2, 3, 4, 5};
        
// Modify array elements using Arrays.setAll
Arrays.setAll(arr, i -> arr[i] * 2);
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