---
title: "[web开发] Flask+Python开发个人博客（一）"
date: 2017-04-05 21:41:47
tags: [dev-in-test, python, 开发个人博客]
categories: [dev-in-test]
---

纯flask实现的一个blog。

<!--more-->

网站的整体效果：
登录页面（也是首页）：
![]([Web开发] Flask+Python 开发个人博客（一）/1.png)

注册页面:

![]([Web开发] Flask+Python 开发个人博客（一）/2.png)

这两个页面都实现了基本的验证，比如：
用户名密码为空

![]([Web开发] Flask+Python 开发个人博客（一）/3.png)

密码不正确：

![]([Web开发] Flask+Python 开发个人博客（一）/4.png)

正确登录后会跳转到系统主页面，长下面的样子：

![]([Web开发] Flask+Python 开发个人博客（一）/5.png)

页面模块：
- Header
- Footer
- 文章模块
点击主页上的模块，或者在任何文章的类别上点击，会进入下面的两个页面：
一个是列在Header上的类别，类别选中会高亮。：

![]([Web开发] Flask+Python 开发个人博客（一）/6.png)

一个是未列在Header上的类别，无高亮：

![]([Web开发] Flask+Python 开发个人博客（一）/7.png)

- 文章分类tag

![]([Web开发] Flask+Python 开发个人博客（一）/8.png)

- 分页模块

参考了Flask上的分页，加上用了Bootstrap的样式

- 新建/修改/删除 文章模块。

用了CK Editor，最好的编辑器之一。

新建：

![]([Web开发] Flask+Python 开发个人博客（一）/9.png) 

修改：

![]([Web开发] Flask+Python 开发个人博客（一）/10.png) 

删除：

![]([Web开发] Flask+Python 开发个人博客（一）/11.png) 

- 文章详页模块

一个是对文章只有浏览权限，没有修改和删除按钮：

![]([Web开发] Flask+Python 开发个人博客（一）/12.png)

一个是文章的作者或者Admin用户组的用户，对文章有full control权限

![]([Web开发] Flask+Python 开发个人博客（一）/13.png) 

- 权限控制模块（包括用户，权限，登录验证，管理员验证等）

![]([Web开发] Flask+Python 开发个人博客（一）/14.png) 

- 项目配置（使用instance）

