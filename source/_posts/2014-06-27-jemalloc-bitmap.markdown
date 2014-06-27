---
layout: post
title: "jemalloc之bitmap"
date: 2014-06-27 23:27
comments: true
categories: 
---

## 1. 数据结构：
结构体bitmap_info_s维护位图的管理信息。<br>
	struct bitmap_level_s {
		/* Offset of this level's groups within the array of groups. */
		size_t group_offset;
	};
	struct bitmap_info_s {
		/* Logical number of bits in bitmap (stored at bottom level). */
		size_t nbits;
		/* Number of levels necessary for nbits. */
		unsigned nlevels;
		/*
		 * Only the first (nlevels+1) elements are used, and levels are ordered
		 * bottom to top (e.g. the bottom level is stored in levels[0]).
		 */
		bitmap_level_t levels[BITMAP_MAX_LEVELS+1];
	};
位图的数据信息使用一个长整型的数组保存。<br>
	typedef unsigned long bitmap_t;
	bitmap_t *bitmap;
	
## 2. 算法分析
bitmap为长整形数组，其中为了能清晰地显示层次，图中分成三层来显示，[]中的下标表示对应level在数组中的起始位置。<br>
数组中使用1表示空闲，0表示占用。每个块为一个长整型（ulong）的变量，保存32bits的数据信息。<br>
<img src="/images/je_bmp.jpg" alt="bitmap" />
以创建一个10000bits的位图为例<br>
Level 0：一共10000bits 表示位图实际使用情况<br>
Level 1：一共313bits，分别表示level 0中每个块的使用情况（第N位对应level 0的第N块的使用情况，若对应的该块的所有32bits都被置上，则level 1中对应的为变为0，否则为1）。<br>
Level 2：一共10bits，分别表示level 1中每个块的使用情况。（最高等级的level肯定只占用了一个块）。<br>
<br>
可以总结为：<br>
创建一个Nbis的位图时<br>
1. 总共需要的level级别为log32(N) <br>
2. 需要使用内存大小为 N + N/32 + N/(32^2) + … + N/(32^max_level) bits，比普通位图多占用了一点内存 <br>
3. 判断整个位图是否是满的，只要比较最高级别level的块是否为0，复杂度为O(1)；普通位图算法需要遍历所有的块来判断，复杂度为O(N/32)。<br>
4. 查找第一个未被使用的位只需要从高到低查找每个level中的第一个空闲位的位置，复杂度为O(log32(N) * 32)；普通位图算法需要遍历所有的块来判断，复杂度为O(N/32)。<br>

Author: chenxiawei@gmail.com<br>