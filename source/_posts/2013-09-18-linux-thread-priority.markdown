---
layout: post
title: "linux线程优先级"
date: 2013-09-18 22:50
comments: true
categories: 
---
## 1. Linux线程优先级范围
Linux定义线程优先级范围在头文件<linux/sched.h></br>
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
	#define MAX_RT_PRIO			MAX_USER_RT_PRIO

	#define MAX_PRIO			(MAX_RT_PRIO + 40)
	#define DEFAULT_PRIO		(MAX_RT_PRIO + 20)

## 1. Linux线程调度策略
Linux定义线程调度策略在头文件<linux/sched.h></br>
	#define SCHED_NORMAL	0
	#define SCHED_FIFO		1
	#define SCHED_RR		2
	#define SCHED_BATCH		3
	/* SCHED_ISO: reserved but not implemented yet */
	#define SCHED_IDLE		5

SCHED_NORMAL即为SCHED_OTHER：采用分时调度策略。<br>
SCHED_FIFO：采用实时调度策略，且先到先服务，一直运行直到有更高优先级任务到达或自己放弃。<br>
SCHED_RR：采用实时调度策略，且时间片轮转，时间片用完，系统将重新分配时间片，并置于就绪队列尾。<br>
SCHED_BATCH：针对批处理进程。<br>
SCHED_IDLE：使用此调度器的进程的优先级最低。在实现CFS时引入。<br>
各个类型的详细说明在[http://www.linuxjournal.com/article/3910](http://www.linuxjournal.com/article/3910)<br>

## 2. Linux线程调度算法

## 3 shell下查看线程优先级
### 1 top命令
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                          
		1 root      20   0  2804 1576 1192 S  0.0  0.3   0:00.97 init                                                                           
		2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd                                                                       
		3 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0                                                                    
		4 root      20   0     0    0    0 S  0.0  0.0   0:00.02 ksoftirqd/0                                                                    
		5 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 watchdog/0                                                                     
		6 root      20   0     0    0    0 S  0.0  0.0   0:00.06 events/0     

PR为线程优先级，范围[0, 40)，实时优先级的线程显示为RT。值越小优先级越高<br>
NI为Nice值，范围[-20, 20)，PR值为初始线程优先级加上Nice值。<br>
PR 和 NI 值都是从 proc文件系统中读取得到 "/proc/[pid]/stat"<br> 
	cat /proc/1/stat
	1 (init) S 0 1 1 0 -1 4202752 4680 154640 23 343 6 136 154 151 20 0 ...
	cat /proc/3/stat
	3 (migration/0) S 2 0 0 0 -1 2216730688 0 0 0 0 0 0 0 0 -100 0  ...

通过man proc 查看stat的第18项和第19项的说明如下<br>
	priority %ld
	(18) (Explanation for Linux 2.6) For processes running a real-time scheduling policy (policy below; see sched_setscheduler(2)), 
	this is the negated scheduling priority, minus one; that is, a number in the range -2 to -100, corresponding to real-time priorities 1 to 99. 
	For processes running under a non-real-time scheduling policy, this is the raw nice value (setpriority(2)) as represented in the kernel. 
	The kernel stores nice values as numbers in the range 0 (high) to 39 (low), corresponding to the user-visible nice range of -20 to 19.

	Before Linux 2.6, this was a scaled value based on the scheduler weighting given to this process.

	nice %ld
	(19) The nice value (see setpriority(2)), a value in the range 19 (low priority) to -20 (high priority).
	
### 2 ps命令
	c@c-desktop:~$ ps -el
	F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
	4 S     0     1     0  0  80   0 -   702 poll_s ?        00:00:01 init
	1 S     0     2     0  0  80   0 -     0 kthrea ?        00:00:00 kthreadd
	1 S     0     3     2  0 -40   - -     0 migrat ?        00:00:00 migration/0
	1 S     0     4     2  0  80   0 -     0 ksofti ?        00:00:00 ksoftirqd/0
	5 S     0     5     2  0 -40   - -     0 watchd ?        00:00:00 watchdog/0
	1 S     0     6     2  0  80   0 -     0 worker ?        00:00:00 events/0
	
这里的PRI为什么和top的PR不一样呢，通过查看procps源代码找到了原因<br>
	In linux procps, the column labeled "PRI" in ps -l is -o opri
	c@c-desktop:~$ ps -e -o pid,pri,opri,intpri,priority,ni,cmd
	  PID PRI PRI PRI PRI  NI CMD
		1  19  80  80  20   0 /sbin/init
		2  19  80  80  20   0 [kthreadd]
		3 139 -40 -40 -100  - [migration/0]
		4  19  80  80  20   0 [ksoftirqd/0]
		5 139 -40 -40 -100  - [watchdog/0]
		6  19  80  80  20   0 [events/0]

procps/output.c中含有具体说明<br>
	// "priority"         (was -20..20, now -100..39)
	// "intpri" and "opri" (was 39..79, now  -40..99)
	// "pri_foo"   --  match up w/ nice values of sleeping processes (-120..19)
	// "pri_bar"   --  makes RT pri show as negative       (-99..40)
	// "pri_baz"   --  the kernel's ->prio value, as of Linux 2.6.8     (1..140)
	// "pri"               (was 20..60, now    0..139)
	// "pri_api"   --  match up w/ RT API    (-40..99)

## 4 shell下修改线程优先级
### 1 nice命令
nice命令可以指定线程加载时的Nice值，来改变线程优先级。<br>
nice -n [priority] [command] <br>
输入的优先级超过19时取19，小于-20时取-20。<br>
	root@c-desktop:~/Code# nice -n -20 ./a.out &
	root@c-desktop:~/Code# ps -o pid,priority,ni,cmd
	  PID PRI  NI CMD
	 2138   0 -20 ./a.out
	 
	root@c-desktop:~/Code# nice -n 19 ./a.out &
	root@c-desktop:~/Code# ps -o pid,priority,ni,cmd
	  PID PRI  NI CMD
	 2145  39  19 ./a.out

### 2 renice命令
renice命令可以改变运行中线程的Nice值，进而改变线程的优先级。<br>
renice -n [priority] -p [pid] <br>
输入的优先级超过19时取19，小于-20时取-20。<br>

	root@c-desktop:~/Code# ps -o pid,priority,ni,cmd
	  PID PRI  NI CMD
	 2145  39  19 ./a.out
	 
	root@c-desktop:~/Code# renice -n -20 -p 2145
	2145: old priority 19, new priority -20
	
	root@c-desktop:~/Code# ps -o pid,priority,ni,cmd
	  PID PRI  NI CMD
	 2145   0 -20 ./a.out

## 5. Reference
(1) [http://superuser.com/questions/286752/unix-ps-l-priority](http://superuser.com/questions/286752/unix-ps-l-priority)<br>
(2) [http://procps.sourceforge.net/download.html](http://procps.sourceforge.net/download.html) 需要翻墙<br>
(3) [http://www.linuxjournal.com/article/3910](http://www.linuxjournal.com/article/3910)<br>

Author: chenxiawei@gmail.com<br>
