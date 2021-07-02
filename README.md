1. 安装Git
Git是目前世界上最先进的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。也就是用来管理你的hexo博客文章，上传到GitHub的工具。Git非常强大，我觉得建议每个人都去了解一下。廖雪峰老师的Git教程写的非常好，大家可以了解一下。Git教程
windows：到git官网上下载,Download git,下载后会有一个Git Bash的命令行工具，以后就用这个工具来使用git。
linux：对linux来说实在是太简单了，因为最早的git就是在linux上编写的，只需要一行代码
sudo apt-get install git
安装好后，用git --version 来查看一下版本

2. 安装nodejs
Hexo是基于nodeJS编写的，所以需要安装一下nodeJs和里面的npm工具。
windows：nodejs选择LTS版本就行了。
linux：
sudo apt-get install nodejs
sudo apt-get install npm
安装完后，打开命令行
node -v
npm -v
检查一下有没有安装成功

顺便说一下，windows在git安装完后，就可以直接使用git bash来敲命令行了，不用自带的cmd，cmd有点难用。

3. 安装hexo
前面git和nodejs安装好后，就可以安装hexo了，你可以先创建一个文件夹blog，然后cd到这个文件夹下（或者在这个文件夹下直接右键git bash打开）。
输入命令
npm install -g hexo-cli
依旧用hexo -v查看一下版本
至此就全部安装完了。

接下来初始化一下hexo
hexo init myblog
这个myblog可以自己取什么名字都行，然后
cd myblog //进入这个myblog文件夹
npm install
新建完成后，指定文件夹目录下有：
node_modules: 依赖包
public：存放生成的页面
scaffolds：生成文章的一些模板
source：用来存放你的文章
themes：主题
** _config.yml: 博客的配置文件**
hexo g
hexo server
打开hexo的服务，在浏览器输入localhost:4000就可以看到你生成的博客了。


问题:
1.修改js-css-image-文章-放到blog中
_config.yml文件配置根路径问题

2.图片问题
post_asset_folder: true
![Untitled.png](Untitled.png)

3.文章可以拥有如下属性：
Setting	Description	Default
layout	layout	post或page
title	文章标题	文档名
date	创建日期	文档的创建日期
updated	修改日期	文档的修改日期
comments	是否开启评论	true
tags	标签	NULL
categories	分类	NULL
permalink	url中的名字	文档名
thumbnail	文章缩略图	NULL
banner	文章缩略图&文章首部banner	NULL

4.Git 无法添加文件夹下文件（如：Hexo下themes的模板文件）

可能是该子文件夹下有.git文件夹导致无法上传
1. 删除子文件夹下.git后，依然无法提交子文件夹下的文件。

2. git rm --cached themes/hueman

3. git add .

4. git commit -m "xxx"

5. git push origin master

如果执行git rm --cached directory 提示fatal: Unable to create
‘xx/.git/index.lock’: File exists. 执行rm -f xx/.git/index.lock后解决
