# Linux进程调度

## 进程分类

**实时进程**

​	需要及时响应。如键盘、鼠标操作。

**普通进程**

​	响应不需要那么及时的进程。如压缩文件，视频编码。

---

## 上下文切换

CPU多个进程，分割时间片的方法

如两个进程P1, P2

P1进程在切换到P2进程前需要先保存其执行的信息，如程序计数器、变量值、程序运行位置等，概括为已执行过的进程指令和数据在相关寄存器与堆栈中的内容，称为上文。正文是指正在执行的指令和数据在寄存器与堆栈中的内容；下文则是待执行的指令和数据在寄存器与堆栈中的内容。

P2进程加载进来，运行P2。

---

## 调度算法

多个进程判断先后执行

**排队，先来的进程优先被执行。**

> FIFO(first in first out)

优势：简单

劣势：必须要等待前面的进程执行完，才能执行后面的进程。如果前一个进程需要的时间很长，则需要等待很久。

**多个进程同时到达时，最短时间进程优先调度**

> STF(shortest time first)

优势：虽然整体执行时间不变，但是整体等待时间下降。

劣势：当长时间的进程先到达时，后面的进程需要等待。

**先完成的进程，优先被调度**

> STCF (shortest time-to-completion first)

优势：具有抢占性，后面来的进程如果完成时间短，则先暂停正在执行的进程，先执行短的进程，完成后再切换回原来的进程。

劣势：长的进程会总被短的进程抢占，会导致响应时间太长。

**把一个较小的单位时间分成很多个时间片，分给每个进程，轮转切换执行**

> RR(round robin)

优势：用户在感觉上是多个进程同时运行的，响应时间短。

劣势：因为上下文切换需要消耗消耗时间，所以总体需要的时间是延长的，并且没有考虑进程重要程度。

---

## 进程队列

一个CPU多个进程，这些进程放在哪里？用什么数据结构存储？

###### 全局队列

多个CPU 全局队列

加锁，释放锁的各种操作，会降低性能。

###### 局部队列

多个CPU，每个CPU都有自己的队列

每个CPU都是从自己的队列去进程然后执行。

---

## 进程优先级

###### 查看进程的优先级ps-l

Linux nice 进程nice值，表示对别的进程的友好程度，即是否会抢占CPU。nice值越高，优先级越低。

###### 怎么设置一个进程的nice值？

nice -n xx(nice值) pro(进程)

renice -n xx -p pro

###### nice值和优先级priority

优先级的值越高，优先级越低。

nice值 -20~19，优先级的值 0~139。

```c
sched.c文件
/*
 * Convert user-nice values [ -20 ... 0 ... 19 ]
 * to static priority [ MAX_RT_PRIO..MAX_PRIO-1 ],
 * and back.
 */
#define NICE_TO_PRIO(nice)	(MAX_RT_PRIO + (nice) + 20)
#define PRIO_TO_NICE(prio)	((prio) - MAX_RT_PRIO - 20)
#define TASK_NICE(p)		PRIO_TO_NICE((p)->static_prio)

/*
 * 'User priority' is the nice value converted to something we
 * can work with better when scaling various scheduler parameters,
 * it's a [ 0 ... 39 ] range.
 */
#define USER_PRIO(p)		((p)-MAX_RT_PRIO)
#define TASK_USER_PRIO(p)	USER_PRIO((p)->static_prio)
#define MAX_USER_PRIO		(USER_PRIO(MAX_PRIO))
```

```c
sched.h文件
/*
 * Priority of a process goes from 0..MAX_PRIO-1, valid RT
 * priority is 0..MAX_RT_PRIO-1, and SCHED_NORMAL/SCHED_BATCH
 * tasks are in the range MAX_RT_PRIO..MAX_PRIO-1. Priority
 * values are inverted: lower p->prio value means higher priority.
 *
 * The MAX_USER_RT_PRIO value allows the actual maximum
 * RT priority to be separate from the value exported to
 * user-space.  This allows kernel threads to set their
 * priority to a value higher than any user task. Note:
 * MAX_RT_PRIO must not be smaller than MAX_USER_RT_PRIO.
 */

#define MAX_USER_RT_PRIO	100
#define MAX_RT_PRIO		MAX_USER_RT_PRIO

#define MAX_PRIO		(MAX_RT_PRIO + 40)
#define DEFAULT_PRIO		(MAX_RT_PRIO + 20)
```

进程分为实时进程和普通进程

实时进程的优先级为0-99。#define MAX_USER_RT_PRIO	100  最大用户交互优先级

普通进程的优先级为100-129。#define MAX_PRIO		(MAX_RT_PRIO + 40)  最大优先级

###### 静态优先级是什么

nice值来决定的优先级，权利在用户，用户在一开始就设定优先级，同时可以更改。

###### 动态优先级是什么

内核来决定，有些进程会超时，要收到惩罚，内核会自动去调低它的优先级，有些进程等待太久，内核就会提高它的优先级。

---

## Linux调度器

###### O(n)调度器

CPU进程队列，队列里放的是带有优先级的队列。

遍历这个队列，找到优先级最高的进程，复杂度是O(n)，效率比较低。

###### O(1)调度器

位运算是最快的，优先级0-139范围。

优先级映射为一个bitmap，在优先级i上有进程，那么第i位则被置为1。bitmap总共有160位，前140位对应的0-139优先级，后20位空闲。运行队列分为active(时间片有剩余)，expired(时间片耗尽)队列，前者存储正在排队等待运行的进程，后者存储已经运行过的进程。

当active中的进程时间片被耗尽时，会移到expired中，active为空时，active与expired交换。

###### CFS调度器

Completely Fair Scheduler   Linux2.6.32版本引入

保证每个进程的相对公平，都能够分到时间片。

新增虚拟运行时间vruntime，受真实运行时间、进程优先度、CPU负载影响。

例如：优先级高的进程 实际运行10ms，虚拟运行时间记录为1ms；优先级低的进程 实际运行10ms，虚拟运行时间记录10ms。

将所有进程按照虚拟运行时间顺序存储在黑红树中，CPU总是调度最左边的进程。通过不断的更新黑红树，来相对公平的分配每个进程运行时间。

时间复杂度O(logN)

