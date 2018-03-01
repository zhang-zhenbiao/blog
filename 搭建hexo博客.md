---
title: 搭建hexo博客 
date: 2017-07-25 17:47 
categories: 博客，hexo
tags: hexo  

---

## 安装环境
	1、git
	2、node.js
	3、hexo

安装git的编译包

	yum -y install gcc zlib-devel openssl-devel perl cpio expat-devel gettext-devel curl autoconf

下载和安装Git

	wget http://soft.itbulu.com/git/git-2.4.6.tar.gz  //下载安装包
	tar -zxvf git-2.4.6.tar.gz						  //解压安装包
	cd git-2.4.6 									  //进入目录
	autoconf && ./configure && make && make install   //编译、安装
<!-- more -->

安装Node.js环境 
	
	yum -y install gcc-c++ openssl-devel              //安装依赖 
	wget http://nodejs.org/dist/node-latest.tar.gz    //下载安装包
	tar -zxvf node-latest.tar.gz					  //解压安装包
	cd node-v8.2.1							          //进入目录
	./configure && make && make install               //编译、安装

安装Hexo

	npm install -g hexo
	hexo init

这里采用npm方式来部署hexo静态博客。会生成一个hexo的文件夹。

安装依赖包

	npm install

生成hexo静态页面

	hexo generate    //或者 hexo g 

生成完毕之后，我们可以看到多了一个public文件夹，这就是我们所谓的静态博客的目录，如果我们需要部署到服务器或者托管平台，只要将hexo生成静态之后，将public文件夹里的文件传上去就可以了。其他系统文件还是放在本地。

本地预览

	hexo server 

然后浏览器中打开http://ip:4000地址，然后就可以看到文件。一般我们直接部署上去后查看一样。

## 写博客

定位到我们的hexo根目录，执行以下命令，就会出现my-first-blog.md文件，编辑这个文件就可以写自己的博客了。

	hexo new 'my-first-blog'


用这个命令的好处是帮我们自动生成了时间

	---
	title: postName #文章页面上的显示名称，一般是中文
	date: 2017-07-25 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
	categories: 默认分类 #分类
	tags: [tag1,tag2,tag3] #文章标签，可空，多标签请用格式，注意:后面有个空格
	description: 附加一段文章摘要，字数最好在140字以内，会出现在meta的description里面
	---

	以下是正文

更多关于Front-matter请参考： [https://hexo.io/zh-cn/docs/front-matter.html](https://hexo.io/zh-cn/docs/front-matter.html)
