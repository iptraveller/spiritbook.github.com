---
layout: post
title: "The Art of Readable Code (1)"
date: 2012-08-27 04:05
comments: true
categories: 
---
“你的代码首先是为人写的，其次才是为计算机写的。”

## 1. 代码的写法应当使别人理解它所需的时间最小化。
Code should be written to minimize the time it would take for someone else to understand it.

### (1) 选择专业的词 (Choose Specific Words)
	GetPage();
	DownloadPage();
	例如希望构造一个表达下载页面的函数，使用Download比Get更专业

send  可替换为 deliver, dispatch, announce, distribute, route <br>
find  可替换为 search, extract, locate, recover <br>
start 可替换为 launch, create, begin, open <br>
make  可替换为 create, set up, build, generate, compose, add, new <br>

### (2) 避免像tmp和retval这样泛泛的名字 (Avoid generic names)
	for (var i = 0; i < v.length; i += 1) retval += v[i] * v[i];
	for (var i = 0; i < v.length; i += 1) sum_squares += v[i] * v[i];
	使用sum_squares能更直观地看出变量的含义。
	
### (3) 为名字附带更多信息 (Attach important details)
	delay(int delay);
	delay(int delay_ms);
	通过添加单位，变量delay_ms就比变量delay含义更清晰。
 
### (4) 使用不会被误解的名字 (Names That Can’t Be Misconstrued)
	CART_TOO_BIG_LIMIT = 10 
	MAX_ITEMS_IN_CART = 10
	最大值，则以MAX_、最小值以MIN_ 开头则不容易被误解其含义。
