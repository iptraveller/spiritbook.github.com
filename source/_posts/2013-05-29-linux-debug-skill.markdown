---
layout: post
title: "Linux调试手段"
date: 2013-05-29 21:19
comments: true
categories: 
---
## 1. 使用gdb
编译时，使用-g 参数把调试信息（函数名、变量名）加到可执行文件中。<br>
###  1.1 启动GDB的方法：
####  (1)、gdb <program>       调试<program> 
####  (2)、gdb <program> core  调试<program>生成的core文件
####  (3)、gdb <program> <PID> 调试运行时的<program>
###  1.2 常用GDB命令:
####  (1)、print [/<fmt>] <expr> (简写p) 显示指定表达式的内容
        <expr> 显示的表达式。可以为全局变量、局部变量、数组名等。
        <fmt>  显示的格式。x 为十六进制格式，c 为字符格式等。
####  (2)、examine [/<len>/<fmt>/<step>] <addr> (简写x) 显示指定地址内容
        <addr> 显示的内存地址的起始地址
        <len> 显示内存的长度
        <fmt> 显示格式。s 为字符串，u 为十六进制。(缺省为十六进制)
        <step> 表示一次显示的字节数。b为单字，h为双字，w为四字。(缺省为四字)
####  (3)、thread <id> 切换到指定线程
        <id> 线程编号
####  (4)、frame <id> 切换到线程中的指定层次的函数
        <id> 函数层次
####  (5)、backtrace [full] 显示当前线程整个栈回溯信息
####  (6)、thread apply all backtrace [full] 显示所有线程的栈回溯信息
####  (7)、info reg 查看当前寄存器信息    
####  (8)、info symbol <addr> 显示函数地址对应的函数名
        <addr> 为函数指针地址 
		
## 2. 使用/proc文件系统

## 3. 使用反汇编
