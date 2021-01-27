+++
title = "使用GCC内置原子操作"
description = "GCC内置了一些原子操作来提升效率，在某些场景可以使用此类操作代替需要使用锁机制的情况提升效率"
author = "David Shu"
date = "2021-01-26T16:20:26+08:00"
tags = ["GCC"]
categories = ["GCC"]
removeBlur = true
[[images]]
  src = "https://static.code-david.cn/blog/ZW83sf.png"
  alt = "gcc atomic"
  stretch = ""
+++

从GCC4.1.版本之后就引入了内置的原子操作函数，可对x86_64架构（除此之外还有其他类型）1、2、4、8字节的integer scalar或pointer使用，可有效减少对锁机制的使用进一步而提升效率，这些函数以`__sync`开头，而在GCC4.7之后的版本，这些函数被替换成了以`__atomic`开头的一系列函数，新编写的代码应该使用这个新的写法，本文将对其使用做简要介绍。

## 主要作用
使用内置的原子操作函数可以部分代替需要锁机制处理的场景，比如高并发场景，锁机制的存在对于高并发是一种效率损失，是为了线程安全之类的原因而做出的取舍，在可以使用内置原子操作的时候建议使用该操作，以提升并发程序的性能

## load、store、exchange等
1. type __atomic_load_n (type *ptr, int memorder)，原子load操作，返回内容指针
2. void __atomic_store_n (type \*ptr, type val, int memorder)，原子store操作，将val放入*ptr
3. type __atomic_exchange_n (type \*ptr, type val, int memorder)，原子exchange操作，将val写入*ptr，返回原来的\*ptr内容
4. bool __atomic_compare_exchange_n (type *ptr, type *expected, type desired, bool weak, int success_memorder, int failure_memorder)原子比较和交换操作，具体参数功能查看下方官网介绍

## 加减法、异或等
1. 先做操作然后取数据（返回值），类似前置++
- Built-in Function: type __atomic_add_fetch (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_sub_fetch (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_and_fetch (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_xor_fetch (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_or_fetch (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_nand_fetch (type *ptr, type val, int memorder)

2. 先取数据（返回值）然后操作，类似后置++
- Built-in Function: type __atomic_fetch_add (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_fetch_sub (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_fetch_and (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_fetch_xor (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_fetch_or (type *ptr, type val, int memorder)
- Built-in Function: type __atomic_fetch_nand (type *ptr, type val, int memorder)

还有其他类型的操作，这里不多做赘述

## memorder参数
以上列出的函数中均有memorder参数，那么这个参数是干嘛用的，可以简单理解为是执行这个原子操作的严格等级，因为越是严格那么就意味着效率损失的越多，为了避免不以必要的效率损失，对严格程度需要做进一步的划分，主要的可用等级如下：

__ATOMIC_RELAXED：最低约束等级，表示没有线程间排序约束

__ATOMIC_CONSUME：官方表示因为C++11的memory_order_consume语义不足，当前使用更强的__ATOMIC_ACQUIRE来实现。

__ATOMIC_ACQUIRE：对释放操作创建线程间happens-before限制，防止代码在操作前的意外hoisting

__ATOMIC_RELEASE：对获取操作创建线程间happens-before限制，防止代码在操作后的意外sinking

__ATOMIC_ACQ_REL：结合了前述两种限制

__ATOMIC_SEQ_CST：约束最强

## 演示
下面代码模拟多线程操作时的原子操作的作用，创建十个线程，每个线程对一个全局变量进行循环加5000000次操作，最后回收线程，输出最终全局变量结果，代码如下
```c++
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

int g_iFlagAtom = 1; // 是否使用原子操作
#define WORK_SIZE 5000000
#define WORKER_COUNT 10
pthread_t g_tWorkerID[WORKER_COUNT];
int g_iSum = 0;

void *thr_worker(void *arg) {
   printf("WORKER THREAD %08X STARTUP\n", (unsigned int)pthread_self());
   int i = 0;
   for (i = 0; i < WORK_SIZE; ++i) {
       if (g_iFlagAtom) {
           __atomic_fetch_add(&g_iSum, 1, __ATOMIC_SEQ_CST); // 原子操作函数
       } else {
           g_iSum ++;
       }
   }
   return nullptr;
}

void *thr_management (void *arg) {
   printf("MANAGEMENT THREAD %08X STARTUP\n", (unsigned int)pthread_self());
   int i;
   for (i = 0;i < WORKER_COUNT; ++i) {
       pthread_join(g_tWorkerID[i], NULL);
   }
   printf("ALL WORKER THREADS FINISHED.\n");
   return nullptr;
}

int main() {
   pthread_t tManagementID;
   pthread_create (&tManagementID, NULL, thr_management, NULL);
   int i = 0; 
   for (i = 0;i < WORKER_COUNT; ++i) {
       pthread_create(&g_tWorkerID[i], NULL, thr_worker, NULL);
   }
   printf("CREATED %d WORKER THREADS\n", i);
   pthread_join(tManagementID, NULL);
   printf("THE SUM: %d\n", g_iSum);
   return 0;
}
```

在使用原子操作时，输出结果如下，可见能准确输出50000000的结果：
```
MANAGEMENT THREAD 70FBE700 STARTUP
WORKER THREAD 707BD700 STARTUP
CREATED 10 WORKER THREADS
WORKER THREAD 6DFB8700 STARTUP
WORKER THREAD 6E7B9700 STARTUP
WORKER THREAD 6FFBC700 STARTUP
WORKER THREAD 6EFBA700 STARTUP
WORKER THREAD 6D7B7700 STARTUP
WORKER THREAD 6CFB6700 STARTUP
WORKER THREAD 67FFF700 STARTUP
WORKER THREAD 6F7BB700 STARTUP
WORKER THREAD 677FE700 STARTUP
ALL WORKER THREADS FINISHED.
THE SUM: 50000000
```

在不使用原子操作时，即把g_iFlagAtom设置为0，得到的结果如下，输出结果是14240946，而不是准确结果，可见多线程下没有原子操作的加法造成了结果错乱：

```
MANAGEMENT THREAD 99AFE700 STARTUP
WORKER THREAD 937FE700 STARTUP
CREATED 10 WORKER THREADS
WORKER THREAD 902FB700 STARTUP
WORKER THREAD 92FFD700 STARTUP
WORKER THREAD 93FFF700 STARTUP
WORKER THREAD 927FC700 STARTUP
WORKER THREAD 98AFC700 STARTUP
WORKER THREAD 90AFC700 STARTUP
WORKER THREAD 91FFB700 STARTUP
WORKER THREAD 992FD700 STARTUP
WORKER THREAD 917FA700 STARTUP
ALL WORKER THREADS FINISHED.
THE SUM: 14240946
```

以上为GCC内置原子操作函数介绍，使用该类函数可减少锁机制的使用而提升效率，在需要极致性能的高并发场景有一定作用。

## 参考资料
【1】[GCC官方介绍](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html)

【2】[旧版__sync系列函数代码演示](https://zhuanlan.zhihu.com/p/32303037)