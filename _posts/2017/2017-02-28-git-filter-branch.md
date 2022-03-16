---
title: 永久删除git中的大文件或者机密文件
categories:
  - Tech
tags:
  - Git
description: removing-sensitive-data-from-a-repository
classes: wide
---

# 永久删除git库中的所有大文件或者机密文件

有时我们忘记了在gitignore中添加大文件或者二进制文件或者记录账号密码的文件，最后又不小心把他们commit了。
如果直接

	git rm some-files

只能使得后面的commit中都不包括这些文件，如果是账号密码文件，别人可以从以前的commit中获取，
如果是大文件，整个git库的大小依然很大
所以解决的办法就是在git仓库中的删除所有的相关文件和记录，采用`git filter-branch`来处理
假设有如下文件，我们需要删除\*.db的文件

![ls](/images/git-filter-branch/git-ls.PNG)

删除前的.git库的大小

![du-before](/images/git-filter-branch/git-du-before.PNG)

>删除匹配\*.db的所有文件

    git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch *.db' --prune-empty --tag-name-filter cat -- --all
>执行结果

![git-filter](/images/git-filter-branch/git-filter.PNG)

>立刻回收空间

	rm -rf .git/refs/original/
	git reflog expire --expire=now --all
	git gc --prune=now
	git gc --aggressive --prune=now

>执行结果

![git-gc](/images/git-filter-branch/git-gc.PNG)

>回收空间后.git库大小

![git-gc](/images/git-filter-branch/git-du-after.PNG)

>最后把改动强制推送到远端

	git push origin --force --all

完成收工~

>**参考链接**

[http://blog.mallol.cn/如何给git仓库瘦身删除大文件.html]( http://blog.mallol.cn/%E5%A6%82%E4%BD%95%E7%BB%99git%E4%BB%93%E5%BA%93%E7%98%A6%E8%BA%AB%E5%88%A0%E9%99%A4%E5%A4%A7%E6%96%87%E4%BB%B6.html )

[https://help.github.com/articles/removing-sensitive-data-from-a-repository/](https://help.github.com/articles/removing-sensitive-data-from-a-repository/ )
