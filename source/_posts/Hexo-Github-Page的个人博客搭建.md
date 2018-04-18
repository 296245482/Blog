---
title: Hexo+Github Page的个人博客搭建
date: 2018-04-18 16:03:21
updated: 2018-04-18 16:03:21
categories: tech
tags: 
	- 个人
description: 本文介绍了本博客的的搭建方法，基于GitHub的免费page服务搭建Hexo项目，有任何问题欢迎留言讨论~
---
## 起因
最近开始各种地方准备春招，投递了不少简历，经历了不少面试，着实感觉自己很多地方的积累还是不够，后端研发或者是Java研发这一块，还有太多的细节没有弄清楚，很多框架的一些原理没有很好的理解，因此决定开一个博客，在平时的学习过程中不断积累，不断总结知识点。


除去hexo配置完成后的“Hello World”文章，个人博客的第一篇文章就以“Hexo+Github Page配置个人博客”为主题了。
## 步骤
配置过程参考了很多现有的网站的一些过程

完成的效果是在本地创建了一份Hexo工程，通过MarkDown创作，每次生成博客网站的内容，通过SSH方式远程部署到自己的USERNAME.GitHub.io的repo上。

Hexo的初步配置参考： [Mac下使用Hexo+Github搭建个人博客](https://www.jianshu.com/p/e5f95eb990ad) 。

初步完成后使用了 [Maupassant](https://www.haomwei.com/technology/maupassant-hexo.html) 主题，该主题很简洁，算是我比较喜欢的类型。

初步配置完成之后，后期有很多主题内评论、语言等配置，参考了[Zhesong的配置过程](https://github.com/handsdirty/hexo_blog) 。

**详细的步骤在上述几个教程中都已经有了完备的描述**

## 发布文章

```
cd [hexo本地工程路径下]
hexo new post '文章标题'
```

打开工程路径下的`/source/_posts/`路径，找到自己刚新建出来的文章，打开编辑内容。
```
hexo clean
hexo generate / hexo g
hexo deploy / hexo d
```
这样的操作即可将新文章内容更新到个人的博客网站上。


PS: 有一种方法可以在deploy之前预览到更新后的样子，执行完 `hexo generate` 后执行 `hexo server / hexo s` ，在浏览器输入 [http://localhost:4000/](http://localhost:4000/) 查看预览结果

## 立个Flag
每隔几天学习完一段时间后，坚持写个人的技术博客，在详细理解之后坚持原创，记录好自己的思路和理解，坚持！
