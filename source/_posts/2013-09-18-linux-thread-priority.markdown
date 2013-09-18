---
layout: post
title: "linux-thread-priority"
date: 2013-09-18 22:50
comments: true
categories: 
---
## 1. 优先级范围
Linux线程优先级范围定义在头文件<linux/sched.h></br>
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

## 2 查看进程优先级方式
### 1 top命令
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                          
		1 root      20   0  2804 1576 1192 S  0.0  0.3   0:00.97 init                                                                           
		2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd                                                                       
		3 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0                                                                    
		4 root      20   0     0    0    0 S  0.0  0.0   0:00.02 ksoftirqd/0                                                                    
		5 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 watchdog/0                                                                     
		6 root      20   0     0    0    0 S  0.0  0.0   0:00.06 events/0     
说明：<br>
PR 和 NI 从 /proc文件系统中读取 "/proc/[pid]/stat"<br> 
	cat /proc/1/stat
	1 (init) S 0 1 1 0 -1 4202752 4680 154640 23 343 6 136 154 151 20 0 ...
	cat /proc/3/stat
	3 (migration/0) S 2 0 0 0 -1 2216730688 0 0 0 0 0 0 0 0 -100 0  ...
	priority %ld

man proc 查看stat文件的具体说明中的第18和第19项<br>

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
	
这里的PRI为什么和TOP的PS值不一样呢，查阅procps源代码后发现如下奥妙<br>
	In linux procps, the column labeled "PRI" in ps -l is -o opri
	c@c-desktop:~$ ps -e -o pid,pri,opri,intpri,priority,ni,cmd
	  PID PRI PRI PRI PRI  NI CMD
		1  19  80  80  20   0 /sbin/init
		2  19  80  80  20   0 [kthreadd]
		3 139 -40 -40 -100  - [migration/0]
		4  19  80  80  20   0 [ksoftirqd/0]
		5 139 -40 -40 -100  - [watchdog/0]
		6  19  80  80  20   0 [events/0]

procps/output.c中有具体说明：<br>
	// "priority"         (was -20..20, now -100..39)
	// "intpri" and "opri" (was 39..79, now  -40..99)
	// "pri_foo"   --  match up w/ nice values of sleeping processes (-120..19)
	// "pri_bar"   --  makes RT pri show as negative       (-99..40)
	// "pri_baz"   --  the kernel's ->prio value, as of Linux 2.6.8     (1..140)
	// "pri"               (was 20..60, now    0..139)
	// "pri_api"   --  match up w/ RT API    (-40..99)

## 3. Reference
(1) [http://superuser.com/questions/286752/unix-ps-l-priority](http://superuser.com/questions/286752/unix-ps-l-priority)<br>
(2) [http://procps.sourceforge.net/download.html](http://procps.sourceforge.net/download.html) 需要翻墙

Author: chenxiawei@gmail.com<br>
