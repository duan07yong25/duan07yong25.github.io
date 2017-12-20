---
layout: post
title: "创建博客命令"
date: 2017-12-20 19:16:06 +0800
comments: true
categories: 
---
1.创建文件
```shell
    rake new_post 
```
2.生成静态站点
```shell
    rake generate
```

3.预览
```shell
    rake preview
```
    此时，可以使用浏览器打开 localhost:4000 查看效果

4.push到远端
```shell
    rake deploy
```
    push 到 GitHub仓库的 master 分支上
