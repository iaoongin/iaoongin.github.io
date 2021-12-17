---
title: 'SVN错误：XXX is scheduled for addition, but is missing'
date: 2019-08-19 17:57:14
category: code
tags:
- svn
toc: true
---
<!-- ![](https://i.loli.net/2020/01/08/khoDBwpMKAb4nJO.jpg) -->
<!-- more -->
# 解决方式

告诉SVN把这个文件退回到之前的状态 "unversioned"，也就是不对这个文件做任何修改，执行完提交即可。

xxx 必须是全路径
```
svn revert xxx --depth infinity
```
如
```
svn revert H:\abc\config\feign --depth infinity
```


# 场景

发生在 `copy to branch or tag`时

# 原因

使用svn标记了将要提交的文件，此时被你删除了，再进行提交的时候，就会找不到。

# 拓展
`svn revert` 顾名思义是对修改过的东西进行回滚。如对已修改未提交的文件回滚；

`深度（depth）`
```
--depth empty：只包含目录自身，不包含目录下的任何文件和子目录。
--depth files：   包含目录和目录下的文件，不包含子目录。
--depth immediates：  包含目录和目录下的文件及子目录。但不对子目录递归。
--depth infinity：  这是默认的，包含整个目录树。
```

`svn status` 查看文件状态。


本文参考    
[SVN错误：but is missing](https://blog.csdn.net/javaNiuLei12/article/details/89497000) 