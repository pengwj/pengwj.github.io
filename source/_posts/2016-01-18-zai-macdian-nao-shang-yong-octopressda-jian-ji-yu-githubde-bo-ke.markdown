---
layout: post
title: "在Mac电脑上用Octopress搭建基于github的博客"
date: 2016-01-18 23:41:17 +0800
comments: true
categories: 
---

#在Mac电脑上用Octopress搭建基于github的博客

*总结网络上大家写下的文章结合自己的搭建过程整理出了一份教程。*


英文较好的同学可以直接阅读英文资料：  
[github](https://help.github.com/categories/github-pages-basics/)   
[Octopress](http://octopress.org)



##1、安装Octopress
在安装之前请自己百度资料安装git和ruby（1.9.3或以上版本）,通过brew安装rbenv然后用rbenv安装ruby。


#####安装brew
先卸载MacPorts

`sudo prot -f uninstall installed`    
  
`sudo rm -fr \`


#####再安装brew

`curl -L http://github.com/mxcl/homebrew/tarball/master | tar xz –strip 1 -C /usr/local`  
 
`export PATH=/usr/local/bin:$PATH`

安装成功后通过brew install查看brew版本


#####安装ruby

`brew install rbenv`   

`brew install ruby-build`   

`rbenv install 1.9.3-p0`   

`rbenv rehash`


#####最后安装Octopress

`git clone git://github.com/imathis/octopress.git octopress`

`cd octopress`

`gem install bundler`

`rbenv rehash`

`bundle install`

`rake install`

#####配置Octopress

编辑 config.yml文件的url,title,subtitle,author。

最好把里面的twitter相关的信息全部删掉，否则由于GFW的原因，将会造成页面load很慢。同理，修改定制文件/source/_includes/custom/head.html 把google的自定义字体去掉。

安装支持新浪微博和Dribbble的Octopress的Greyshade主题

我现在用的就是greyshade主题 http://www.devpeng.com

 `cd octopress`

` git clone git@github.com:allenhsu/greyshade.git .themes/greyshade`

` echo "\$greyshade: color;" >> sass/custom/_colors.scss //替换 color 为自定义的链接高亮颜色`

` rake "install[greyshade]"`

在_config.yml中加入
weibo_user: xsxiang # 微博数字 ID 或域名 ID
dribbble_user: 
weibo_share: true # 是否开启微博分享按钮

关于greyshade主题的头像问题，有两种途径可以设置头像

在_config.yml文件中设置一个email，然后到gravatar网站上添加该email并上传一张头像
将需要使用的图片放到/source/images下。然后把/source/_includes/header.html文件中的img替换成 《img alt=“Profile Picture” src=“/images/tx.png” style=“width:160px;”》

######高能:如果你要映射到自己的域名上面请进行如下操作(如果不做映射请不要进行如下操作)
`cd ~/octopress`

`vim CNAME`

然后在CNAME文件中填写你的域名，比如我的就是(不要加http前缀)
`www.devpeng.com`

#####配置Disqus插件

Disqus是octopress内置的comments功能，编辑config.yml文件可以打开该功能，找到以下内容修改

Disqus comments
disqus_short_name: 
disqus_show_comment_count: false

填入注册[disqus](https://disqus.com/home/explore/)账号的名称，并将false修改为true。【disqus要和自己的username.github.com关联上】

##2.配置github相关
#####在本机创建ssh
`cd ~/.ssh`

`ssh-keygen -t rsa -C 你注册github时的email`

`弹出Enter file in which to save the key (/Users/twer/.ssh/id_rsa):直接按空格`

`弹出Enter passphrase (empty for no passphrase):输入你github账号的密码。Enter same passphrase again:再次输入你的密码。`

打开~/.ssh下的id_rsa.pub文件复制里面的全部内容。
登陆github，选择Account Settings-->SSH Public Keys 添加ssh，把剪切板的内容(全部复制)复制到key输入框内直接保存。

测试shh:
`ssh git@github.com`

输出
PTY allocation request failed on channel 0
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.

代表成功

#####建立一个仓库
登陆[github](https://github.com/new)创建一个仓库(高能：注意不要往里面添加任何内容，包括README.md文件都不要添加),仓库名称为username.github.io比如我的是[pengwj.github.io](pengwj.github.io)

##3.部署博客到github
利用octopress的一个配置rake任务来自动配置上面创建的仓库：可以让我们方便的部署GitHub page。在终端输入如下命令：

`rake setup_github_pages`

弹出之后输入https://github.com/your_username/your_username.github.io,我的是https://github.com/pengwj/pengwj.github.io  

输入以下命令部署博客

`rake generate`

`rake deploy`

如果无法push到仓库的master分支，尝试在项目目录的.git/config中添加
[branch "master"]
 remote = origin
 merge = refs/heads/master

博客的source需要单独提交，执行如下命令就可以将source提交到仓库的source分支下

`git add .`

`git commit -m 'Initial source commit'`

`git push origin source`

部署前可以在本地预览，输入rake preview之后在浏览器输入http://localhost:4000/来访问

##4.写博客   
通过命令     
`rake new_post["myTitle"]    `

文章生成在目录下的source/_posts目录下。文章是markdown格式的。可以通过 Mou 软件来编辑保存。

关于markdown的格式可以参考这篇文章:[http://wowubuntu.com/markdown/](http://wowubuntu.com/markdown/)

写完后就可以部署更新文章到github上了

`rake generate`

`git add .`

`git commit -am "Some comment here." `

`git push origin source`

`rake deploy`

##5.映射到自己的域名（需要先按照步骤一中的操作配置好CNAME文件）


#####获取我们在github上托管page页面的ip

在终端中输入：
`ping pengwj.github.io`

终端输出：

`PING github.map.fastly.net (103.245.222.133): 56 data bytes`

其中，`103.245.222.133` 就是我的ip

#####打开我们的域名解析中心
我的域名 -> 域名解析 -> 解析设置 -> 添加解析
添加两条记录：

|记录类型   | 主机记录    | 解析线路  | 记录值             |
|---------|:---------:|-------: |------------------:|
|A  	  |@           | 默认      | 103.245.222.133   |
|A  	  |www         | 默认      | 103.245.222.133   |

保存后等待一段时间，即可通过自己的域名访问博客了。




参考文章:[http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)

