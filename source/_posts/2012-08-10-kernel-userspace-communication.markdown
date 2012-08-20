---
layout: post
title: "LINUX 进程内核通信"
date: 2012-08-10 20:44
comments: true
categories: 
---

## 1. netlink socket
netlink socekt是一种用于在内核态和用户态进程之间进行数据传输的特殊的IPC。它通过为内核模块提 
供一组特殊的API，并为用户程序提供了一组标准的socket接口的方式，实现了一种全双工的通讯连接。
类似于TCP/IP 中使用AF_INET 地址族一样，netlink socket使用地址族AF_NETLINK。
内核头文件
    #include <linux/netlink.h>
	#include <net/netlink.h>
	#include 
进程头文件	
	#include <linux/netlink.h>
  A. 添加新的netlink类型 <br>
  B. 使用通用netlink类型通信(generic netlink)
## 2. 系统调用
## 3. ioctl接口
## 4. proc fs
