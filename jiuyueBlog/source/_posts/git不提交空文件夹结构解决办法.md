---
title: git不提交空文件夹结构解决办法
date: 2016-12-19 15:21:46
tags: git
---
git 默认忽略空目录结构,所以要在每个空目录下添加.gitignore文件 让git检测到空目录

cd到git目录下 执行以下命令
```
$ find . -type d -empty -exec touch {}/.gitignore \;
```
