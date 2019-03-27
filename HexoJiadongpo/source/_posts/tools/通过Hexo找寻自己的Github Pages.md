---
title: 通过Hexo找寻自己的Github Pages
date: 2018-10-09 19:00:00
tags: [tools.博客]
categories: [tools,博客]
---
# 通过Hexo找寻自己的Github Pages
## 一、项目初始化
1、 创建与用户名同名的项目。
    比如你的github用户名为xxx，即创建项目名为:xxx.github.io的项目。
    
2、安装Hexo
命令行安装hexo服务端和客户端
npm install hexo-server —save 
npm install hexo-cli -g

初始化项目,名称为blog
hexo init blog
cd blog
npm install
hexo server
`
3.Hexo常用命令
$hexo clean
$hexo generate
$hexo server    这个命令用于测试：http://localhost:4000/
$hexo deploy


## 二、配置主题
下载主题
$ cd hexo
$ git clone https://github.com/theme-next/hexo-theme-next themes/next

## 三、相关配置

## 四、域名绑定

## 五、修改默认分支

## 六、参考
参考：[Github page 帮助](https://help.github.com/categories/github-pages-basics/) 




