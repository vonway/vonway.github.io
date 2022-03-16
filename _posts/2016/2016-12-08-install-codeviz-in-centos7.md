---
title: CentOS7安装使用codeviz
categories:
  - Blog
tags:
  - centos
  - codeviz
description: install codeviz in centos7
---

# codeviz安装记录和使用实例

# 1. 简介<a id="orgheadline1"></a>

codeviz是一个分析c/c++项目函数调用关系的工具，基本思路是给gcc打上补丁，
用这个打了补丁的gcc编译要分析的c/c++项目，编译过程中会把函数调用关系dump成文本文件(.cdepn)。
再用perl脚本(genfull和gengraph)把调用关系文件转成graphviz的dot文件，并用graphviz转成图片。

```
yum -y install perl graphviz

# yum 没有找到包，可以手动安装
http://www.graphviz.org/Download.php

tar xvf 下载的包名
cd 解压后的文件夹
./configure && make && sudo make install

```

# 2. 安装<a id="orgheadline6"></a>

## 2.1 获取源码<a id="orgheadline2"></a>

- codeviz最新版1.0.12

```
# 下面的地址已经失效，可以使用github的地址下载
# wget http://www.csn.ul.ie/~mel/projects/codeviz/codeviz-1.0.12.tar.gz
https://github.com/petersenna/codeviz

```

- codeviz-1.0.12匹配的gcc是3.4.6和4.6.2，选择4.6.2

```
# 使用中科大的GNU镜像服务器
wget http://mirrors.ustc.edu.cn/gnu/gcc/gcc-4.6.2/gcc-4.6.2.tar.gz

```

## 2.2 编译安装<a id="orgheadline5"></a>

解压codeviz: `tar xzvf codeviz-1.0.12.tar.gz`, codeviz官方主页的安装说明

```
cd codeviz-1.0.12
./configure && make && make install
```

运气好的话，可以一次成功；不过凡人运气有限，尽量不用在小事上。下面说明如何一步一步安装。

### 2.2.1 编译gcc-4.6.2<a id="orgheadline3"></a>

```
# 编译gcc的依赖
yum install gmp-devel mpfr-devel libmpc-devel glibc-devel glibc-devel.i686

# yum 包找不到，手动安装如下
	# gmp-5.0.1
        https://gmplib.org/
	# mpc-1.0.3
        http://www.multiprecision.org/index.php?prog=mpc&page=download
    # mpfr-3.1.4
        http://www.mpfr.org/
# 安装方式都和安装graphviz的一样，tar xxx ; cd xxx; ./configure && make && sudo make install

# 如果codeviz-1.0.12/compilers下没有gcc-4.6.2，安装脚本会从gnu/gcc官网下载，很慢！
# 所以要事先从国内服务器下载
cp gcc-4.6.2.tar.gz codeviz-1.0.12/compilers
cd codeviz-1.0.12/compilers
# 先编译，排除错误编译通过后，再make install
# 执行下面一句之前，请先按照后面的要求修改install_gcc-4.6.2.sh
./install_gcc-4.6.2.sh /usr/local/gcc-graph compile-only
cd gcc-graph/objdir
sudo make install
```

install\_gcc-4.6.2.sh默认使能共享库 `--enable-shared` 并自举编译 `make bootstrap`

- 共享库要指明编译参数 `-fPIC`
- 自举编译多半不能编译通过，所以要去使能自举编译 `--disable-bootstrap`
- 64bit的系统找不到gnu/stub-32.h头文件 `--disable-multilib`
- 为了加快编译，可以指明用4个线程编译 `make -j 4`

根据这四点，给install\_gcc-4.6.2.sh打补丁，如下：

```
diff -pruN install_gcc-4.6.2-old.sh install_gcc-4.6.2.sh
--- install_gcc-4.6.2-old.sh
+++ install_gcc-4.6.2.sh
@@ -32,8 +32,8 @@
cd ../objdir

# Configure and compile
-../gcc-4.6.2/configure --prefix=$INSTALL_PATH --enable-shared --enable-languages=c,c++ || exit
-make bootstrap
+CFLAGS="-fPIC" ../gcc-4.6.2/configure --prefix=$INSTALL_PATH --disable-bootstrap --enable-shared --enable-languages=c,c++ --disable-multilib || exit
+make -j 4

RETVAL=$?
PLATFORM=i686-pc-linux-gnu
```

也可以手动修改install_gcc-4.6.2.sh文件，增加CFLAGS="-fPIC"  --disable-bootstrap  --disable-multilib

### 2.2.2 安装codeviz-1.0.12<a id="orgheadline4"></a>

codeviz绘制调用图用到两个perl脚本genfull和gengraph，其中，genfull依赖于perl-DB\_File

```
yum -y install perl-DB_File
// yum 安装找不到的话可以下载 https://github.com/vonway/raw_libary/blob/master/perl-DB_File-1.830-6.el7.x86_64.rpm
// 执行rpm -ivh perl-DB_File-1.830-6.el7.x86_64.rpm 进行安装
cd codeviz-1.0.12
./configure
sudo make install-codeviz
```

# 3. 使用<a id="orgheadline9"></a>

## 3.1 codeviz gcc-4.6.2编译源文件<a id="orgheadline7"></a>

- 测试源文件

```

	#include <stdio.h>

	void test1();
	void test2();
	void test3();

	int main()
	{
		test1();
		test2();

		return 0;
	}

	void test1()
	{
	}

	void test2()
	{
		test2();
	}

	void test3()
	{
	}

```

- 编译生成cdepn

```
/usr/local/gcc-graph/bin/gcc -c test.c

```

- 生成graph

```
# 生成fullgraph
genfull -f test.c.cdepn
# 生成函数main的graph文件main.ps
gengraph -f main

```

- 打开ps可以看到函数调用图

![graph](/images/callGraph.png)

## 参考链接

[http://mstry.github.io/2016/05/15/codeviz-install-and-use/](http://mstry.github.io/2016/05/15/codeviz-install-and-use/)

