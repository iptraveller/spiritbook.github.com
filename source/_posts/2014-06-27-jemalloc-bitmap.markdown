---
layout: post
title: "jemalloc之bitmap"
date: 2014-06-27 23:27
comments: true
categories: 
---

## 1. 数据结构：
位图的数据信息使用一个长整型的数组保存。这和我们常见的位图处理方式是一样的。<br>
	typedef unsigned long bitmap_t;
	bitmap_t *bitmap;
	
结构体bitmap_info_s维护位图的管理信息。这是和常见位图区别的地方，jemalloc把位图分为多个level，使用一个管理结构保存每个level在实际bitmap数据中的起始位置。<br>
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

## 2. 算法分析
bitmap为长整形数组，为了能清晰地显示层次，图中分成三层来显示，[]中的下标表示对应level在数组中的起始位置。<br>
数组中使用1表示空闲，0表示占用。每个块为一个长整型（ulong）的变量，保存32bits的数据信息（本文中均以32位平台，ulong为4字节为例）。<br>
<img src="/images/je_bmp.jpg" alt="bitmap" /><br>
以创建一个10000bits的位图为例<br>
Level 0：一共10000bits 表示位图实际使用情况，10000bits除以每个块32bits并向上取整，为313个块<br>
Level 1：一共313bits，分别表示level 0中每个块的使用情况（第N位对应level 0的第N块的使用情况，若level 0对应的该块的所有的32bits都被置上，则level 1中对应的位为0，否则为1）。<br>
Level 2：一共10bits，分别表示level 1中每个块的使用情况（对应方式同上），最终最高等级的level肯定只占用了一个块。<br>
<br>
 
可以总结为：<br>
创建一个N bis的位图时<br>
1. 总共需要的level级别为log32(N) 的值向上取整<br>
2. 需要使用内存大小为 N + N/32 + N/32^2  + … + N/32^max_level bits，普通位图只占用N bits的内存，因此该做法比普通位图算法多占用了约N/31 bits左右的内存 <br>
3. 设置位图中某个位时，先设置level 0中对应位置的位，若该位置对应的块全部被使用，则修改level 1中对应的位为0，依次类推到最高level。<br>
4. 取消位图中某个位时，先取消level 0中对应位置的位，若该位置对应的块从全满变成不满时，则修改level 1中对应的位为1，依次类推到最高level。<br>
5. 判断整个位图是否是满的，只要比较最高级别level的块是否为0，复杂度为O(1)；普通位图算法需要遍历所有的块来判断，复杂度为O(N/32)。<br>
6. 查找第一个未被使用的位只需要从高到低查找每个level中的第一个空闲位的位置，复杂度为O(log32(N) * 32)；普通位图算法需要遍历所有的块来判断，复杂度为O(N/32)。<br>
<br>
由于jemalloc算法中需要经常判断某个chunk中的内存块是否被全部分配，或者分配第一个未被分配的内存，因此该算法可以极大优化性能。对于平时用到的位数不多的情况下，该算法则会显得太臃肿。。<br>

### 3. References
(1) [http://www.canonware.com/jemalloc/](http://www.canonware.com/jemalloc/) <br>

Author: chenxiawei@gmail.com<br>