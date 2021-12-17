---
title: git push 本地项目到远程仓库
date: 2019-08-16 09:22:44
category: code
tags:
- git
toc: true
---
<!-- ![](https://i.loli.net/2019/11/26/IOCUvxrVcesMJab.jpg) -->
<!-- more -->

# 1.进入项目目录

```
cd H:\blog
```

# 2.初始化仓库

```
git init
```

# 3.添加所有文件到仓库

```
git add -A
```

# 4.提交

```
git commit -m "init"
```

# 5.建立好远程仓库

```
https://github.com/iaoongin/iaoongin.github.io.git
```

# 6.配置远程仓库

## http方式
### 此方式需要输入密码
```
git remote add origin https://github.com/iaoongin/iaoongin.github.io.git
// origin 是远程仓库的别名，用于代替git仓库地址
```

## ssh方式
### 此方式需要生成秘钥
```
git remote add origin git@github.com:iaoongin/iaoongin.github.io.git
```
```
ssh-keygen -t rsa -C "这里换上你的邮箱"
```
#### 生成秘钥,如下图 
![生成秘钥](/img/git/ssh-key.png) 
#### 配置远程仓库key
![配置远程仓库key](/img/git/ssh-key-2.png) 

# 7.开始推送

```
git push origin master:source
// git push <远程主机名> <本地分支名>:<远程分支名>


```

## 分支说明
```
// 本地默认初始为master分支
// git branch -a 查看所有本地分支
// git branch 查看当前分支
// git branch abc 创建分支abc
// git checkout abc 切换abc分支
```

