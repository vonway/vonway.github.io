---
title: markdown常用语法
categories:
  - Markdown
tags:
  - markdown
description: markdown syntax description
classes: wide
---

# markdown语法记录和实例展示

### 标题

	# header1
	## header2
	###### header6

### 区块引用

	> 这是引用的部分

> 这也是引用的部分

	> 引用中也可以使用其他的markdown语法

> 这也是**引用**的部分


### emoji

	vonway :+1: This blog looks great ! :shipit:

vonway :+1: This blog looks great ! :shipit:

emoji list:

<http://www.webpagefx.com/tools/emoji-cheat-sheet/>


### 列表

	* + - 开头的无序列表

* zhangsan
* lisi
* wangwu

1\. 2\. 3\. \.\.\. 为有序列表

1. hello
2. world
3. !


### 代码

四个空格或者一个制表符开头

	#include <stdio.h>
	int main(){
		printf("hello world!\n");
		return 0;
	}

相当于html中的:

	<pre><code>
		#include <header.h>
	</code></pre>

如果是一句中的部分代码：

	哈哈哈后面的是代码`#include <iostream>`在后面的没有代码

哈哈哈后面的是代码`#include <iostream>`在后面的没有代码


### 分割线

	三个及以上的*或者-或者_的一行，表示分割线
	如：
	***
	---
	_____

---------------------

啊

***

哈

-----

### 链接

行内式

链接文字用[]括起来，后面的()中卫网址或者同一个主机的资源

	这是一个链接[click](www.vonway.cn)
	还可以加上可选标题。如：
	这是一个有标题的链接[click](www.vonway.cn "这是标题")

[click](http://vonway.cn/)

参考式
就是将行内式的网址，换成一个id

	这是一个链接[点我][1]
	[1]: www.vonway.cn "home"

这是一个链接

	[click](http://vonway.cn)

[click](http://vonway.cn)

这是另一个链接

	[about](http://vonway.cn/about)

[about](http://vonway.cn/about)

这是另一个相同的链接

	[about] [2]

[about] [2]

	[1]: http://www.vonway.cn
	[2]: http://www.vonway.com/about
	[about]: http://www.vonway.com/about


[1]: http://www.vonway.cn
[2]: http://www.vonway.com/about
[about]: http://www.vonway.com/about


自动链接

	<haisayu@gmail.com>

<haisayu@gmail.com>

### 强调

	*或者_包围的斜体
	**或者__ 加粗
	~~ 划线

**strong**

*em*

__ss__

_ll_

~~ deleted ~~

### 图片

和链接很像

	![图片的替代文字](图片地址)
	也有行内式和参考式
	![doge](/images/doge.png)

![doge](/images/doge.png)

### 转义

	\可以转移的符号：

	\   反斜线
	`   反引号
	*   星号
	_   底线
	{}  花括号
	[]  方括号
	()  括弧
	#   井字号
	+   加号
	-   减号
	.   英文句点
	!   惊叹号

### 换行

在一行的末尾加上两个空格后回车，相当于换行`<br>`,不用利用空行换行了~

