---
title: Hexo博客部署到阿里云
date: 2020-04-14 08:27:58
tags: 
	- hexo
	- aliyun
categories: 博客
---
## 1.Hexo博客准备

hexo博客基于nodeJS的静态博客,支持md格式,可以部署到github,也可以部署到其它服务器,这里记录如何部署到阿里云.

### 1.1 下载安装nodejs
下载地址:  http://nodejs.cn/download/

我是win10的64位系统,就选择 
Windows 安装包 (.msi) *64位

安装地址:  F:\Program Files\nodejs (随意)

测试: window+R,输入cmd,打开命令提示符窗口，输入：(提示版本号为安装成功)

    node -v
    npm -v



### 1.2 安装cnpm
装个cnpm的速度会快一点

    npm install -g cnpm --registry=https://registry.npm.taobao.org

测试:cnpm -v  如果有出现版本号证明安装成功
### 1.3 安装hexo

使用cnpm安装


    cnpm install -g hexo-cli


等待安装完成,hexo -v 验证版本

### 1.4 hexo初始化

新建一个文件夹用来用来存放博客

我存放在 C:\Users\Administrator\blog (随意)

(1) 点击地址栏输入 cmd 回车
![](http://image.jiruby.cn/2020-04-14%20100003.jpg)

(2) 初始化 hexo init
![](http://image.jiruby.cn/2020-04-14%20100933.jpg)

### 1.5 hexo基本命令

(1)清空已经生成的静态文件

    hexo clean

(2)生成静态文件

    hexo g

(3)启动本地部署 

    hexo s

 预览地址
http://localhost:4000 (在命令行最后有提示)

(4)推送到远端 (后面部署到阿里云需要用到)

    hexo d

(5)创建新的博客

    hexo new "我的第一篇博客"

## 2.部署到阿里云

### 2.1 购买ECS

最低配的就够了,操作系统 CentOS 7.4 64 位

### 2.2 购买域名并备案 

网站名,网站内容简介最好按引导的填,不然第一关在阿里都不容易过审

这个审批流程需要一段时间

### 2.3 搭建环境

**(1)安装nginx**

安装gcc gcc-c++

    yum install -y gcc gcc-c++

安装PCRE库

    cd /usr/local/

    wget http://downloads.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz

    tar -xvf pcre-8.37.tar.gz

    cd pcre-8.37

    ./configure

    make && make install

    pcre-config --version

安装 openssl 、zlib 、 gcc 依赖

    yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
 
安装nginx

    cd /usr/local/

    wget http://nginx.org/download/nginx-1.17.9.tar.gz

    tar -xvf nginx-1.17.9.tar.gz

    cd nginx-1.17.9

    ./configure

    make && make install

修改配置文件 (当前位置 ~)

    cd /usr/local/nginx/conf

    cp nginx.conf nginx.conf.back

	vim nginx.conf

使用i(nsert) 插入 修改 root 后的地址 /home/www/website;

![](http://image.jiruby.cn/2020-04-14%20105636.jpg)

**(2)安装git及配置仓库**

安装git及新建git用户

    yum install git

	adduser git

	chmod 740 /etc/sudoers

	vi /etc/sudoers

编辑git 用户权限

添加一句  git ALL=(ALL) ALL
	![](http://image.jiruby.cn/2020-04-14%20110211.jpg)

保存退出

执行以下指令文件夹权限恢复,并设置git用户密码

    chmod 400 /etc/sudoers

	sudo passwd git

切换git用户并且建立密钥

    su git

	cd ~

	mkdir .ssh

	cd .ssh

	vi authorized_keys

复制本地密钥(C:\Users\Administrator\\.ssh\id_rsa.pub)中的内容到云端

修改密钥文件夹和密钥文件权限

    chmod 600 ~/.ssh/authorized_keys

	chmod 700 ~/.ssh

创建git仓库

	cd ~

	git init --bare blog.git

	vi ~/blog.git/hooks/post-receive

文档输入以下 ,然后保存退出 (钩子部署)

	git --work-tree=/home/www/website --git-dir=/home/git/blog.git checkout -f

赋予post-receive 可以执行权

	chmod +x ~/blog.git/hooks/post-receive

新建/home/www/website文件夹,用于存放静态网页
在root用户下执行，所限先su root切换为root账户

	su root

	cd /home

	mkdir www

	cd www

	mkdir website

	chmod 777 /home/www/website

	chmod 777 /home/www

在本地电脑输入,连接成功

	ssh -v git@服务器的公网ip

修改Hexo (C:\Users\Administrator\blog\_config.yml)里的本地配置文件


repo: git@这里改为服务器公网IP:/home/git/blog.git

	deploy:
 		type: git
 		repo: git@服务器公网ip:/home/git/blog.git
 		branch: master

**(3)在/etc/init.d/路径下添加脚本文件，名称为nginx，内容如下**

	#!/bin/bash
	#Startup script for the nginx Web Server
	#chkconfig: 2345 85 15
	nginx=/usr/local/nginx/sbin/nginx
	conf=/usr/local/nginx/conf/nginx.conf
	case $1 in 
	start)
	echo -n "Starting Nginx"
	$nginx -c $conf
	echo " done."
	;;
	stop)
	echo -n "Stopping Nginx"
	killall -9 nginx
	echo " done."
	;;
	test)
	$nginx -t -c $conf
	echo "Success."
	;;
	reload)
	echo -n "Reloading Nginx"
	ps auxww | grep nginx | grep master | awk '{print $2}' | xargs kill -HUP
	echo " done."
	;;
	restart)
	$nginx -s reload
	echo "reload done."
	;;
	*)
	echo "Usage: $0 {start|restart|reload|stop|test|show}"
	;;
	esac

赋予执行权

	chmod +x nginx

可以用指令:

启动service nginx start

停止service nginx stop

重启service nginx reload

### 2.4 测试

**(1)测本地git用户可正常登录云端**

	ssh -v git@服务器的公网ip

**(2)退出后,在blog文件目录下执行,直接推送hexo博客到云端**

	hexo clean
	hexo g
	hexo d

输入域名查看博客正常显示,搞定!~




 







    





















