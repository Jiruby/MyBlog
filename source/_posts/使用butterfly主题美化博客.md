---
title: 使用butterfly主题美化博客
date: 2020-04-14 12:28:47
tags: 
	- butterfly
categories: 博客
---
## 1.下载并更换主题
### 1.1 下载
在博客的根目录

	git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

### 1.2 更换主题

修改博客根目录下的配置文件 _config.yml

	# Extensions
	## Plugins: https://hexo.io/plugins/
	## Themes: https://hexo.io/themes/
	theme: Butterfly


## 2.配置buttfly主题

### 2.1 新建_data文件夹,移动配置文件

在hexo博客根目录下的source里新建文件夹
![](http://image.jiruby.cn/2020-04-14%20123609.jpg)

将Butterfly根目录下的 _config.yml复制到  _data下,重命名为 butterfly.yml

![](http://image.jiruby.cn/2020-04-14%20124618.jpg)

### 2.2 修改配置 butterfly.yml--导航

**修改导航栏为中文**

	menu:
	  首页: / || fa fa-home
	  时间轴: /archives/ || fa fa-archive
	  留言板: /message/ || fa fa-coffee
	  标签: /tags/ || fa fa-tags
	  分类: /categories/ || fa fa-folder-open
	  关于: /about/ || fa fa-heart
	  列表||fa fa-list:
	    - 音乐 || /music/ || fa fa-music
	    - 视频 || /movies/ || fa fa-film

测试页面发现除了首页和时间轴,其它页面都无法访问

下面我们一个个创建

**(1)标签页**

	hexo new page tags

修改 source/tags/index.md文件

	---
	title: 标签
	date: 2020-04-14 14:49:24
	type: "tags"
	---

测试通过,
剩下的也就类似了


md 中多个标签的格式

	tags: 
	- hexo
	- aliyun

**(2)分类页**

	hexo new page categories


**(3)留言板**

	hexo new page message

**(4)关于**
	
	hexo new page about

### 2.2 文章内部设置

**(1)代码部分**

代码背景色 

	highlight_theme: darker

支持代码复制

	highlight_copy: true 

代码框展开

	highlight_shrink: true

**(2)文章相关**

字数统计

在hexo根目录下执行以下,使用cnpm不容易失败,官方用的是npm

	cnpm install hexo-wordcount --save

配置 butterfly.yml:

	wordcount:
	  enable: true
	  post_wordcount: true
	  min2read: true
	  total_wordcount: true




	



















