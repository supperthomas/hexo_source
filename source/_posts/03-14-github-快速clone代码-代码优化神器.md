title: '03-14 [github][快速clone代码]代码优化神器'
author: Thomas Li
abbrlink: 3349866602
date: 2020-03-14 10:58:35
tags:
---
1. 用gitee下载

![filename already exists, renamed](/images/pasted-15.png)




选择github地址，之后，就可以用gitee 下载源码，
之后改下remote用下面的命令，或者直接修改.git/config里面的文件


　1.git remote 不带参数，列出已经存在的远程分支

　2.git remote -v 列出详细信息，在每一个名字后面列出其远程url，此时， -v 选项(译注:此为 –verbose 的简写,取首字母),显示对应的克隆地址。

  3.git remote add url   添加一个远程仓库