+++
date = "2015-11-27T11:49:14+08:00"
draft = false 
title = "Jekyll快速开始"
tags = ["Tools"]

+++

# Jekyll ——简单的博客生成工具

[Jekyll] [1]是一个开源的博客生成工具，类似WordPress，但又完全不同。正如官网介绍“Transform your plain text into static websites and blogs.”
jekyll只能生成静态的页面，无需数据库的支持。另外可以配合第三方服务,例如[Disqus] [2] （提供博客评论服务）。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。

### 安装

1. 首先确保有正确的环境
*   Ruby
*   RubyGems
*   Mac or Linux  windows没有测试 
*   NodeJS 或者其他支持JavaScript的平台也可以（自己在这里吃了亏）

2. 通过RubyGems安装jekll
``` $ gem install jekyll```
安装后就可以开始搭建blog了
 
### 使用Jekyll Bootstrap工具

1. 为了方便我是直接选择了用[Jekyll Bootstrap] [3] 简单快捷还有很多模板可以使用或参考。
首先要有一个github的账户，然后新建仓库命名为USERNAME.github.io （USERNAME换成自己的用户名）
在本地fork代码安装Jekyll-Bootstrap

        $ git clone https://github.com/plusjade/jekyll-bootstrap.git XXX        #克隆git到本地命名为XXX
        $ cd XXX     #进入该目录
        $ gi t remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
        $ git push origin master
        	
这时你已经有了自己博客了，怎么样很简单吧。
运行` $ jekyll serve `
测试成功

### 发表文章

#### 1. 新建一个帖子（post） 

` $ rake post tilte="Hello World" `jekyll会自动新建一个文件默认名是时间。

#### 2. 新建一个页面（pages）

> 除了文章外还能新建页面 
> `$ rake page name="about.md"  `

> 新建嵌套页面
> `$ rake page name="pages/about" `

#### 3. 发布
写完文章后就要发布了，通过git可以很容易的完成。

    $ git add .
    $ git commit -m "Add new content"
    $ git push origin master

#### 4. 个性化
用人家的模板有了自己的博客对一般不想折腾的够了，但是有些朋友会有自己的创意想改动一些。Jekyll Bootstrap
支持定制。除了有很多主题供选择外有一些具体的设置。如设置博客地址格式、开启评论功能、设置基本路径、提供分析等等。总之个人觉得对一般的需求都可以满足。想要更高级的设置还可以看到Jekyll Bootstrap提供了各种可用的API，可以调用让自己的博客更加漂亮。

#### 5. 补充
不知怎么直接调用jekyll的官方引擎_jekyll serve_生成静态html,结果显示某些代码支持的不好。有网友说是官方引擎和github pages的自己的引擎有些差别，markdown的解析器不同,导致用官方jekyll测试生效，推到github pages后却不会生效。所以为了发布到github上文章正确可以在本地搭建github pages本地环境预览一下。[搭建方法][https://help.github.com/articles/using-jekyll-with-pages] 试了下发现果然用github pages的引擎比较好

### 总结

用Jekyll Bootstrap真的很方便。第一次写点儿算是教程的东西大部分来自网络。其实就是给自己折腾做个纪念吧，毕竟刚刚接触这个很好奇也很有动力。
感谢Jekyll Bootstrap的开发者们，在模板介绍页面开发者们提到这是个未完成的模板，想要完成它需要大家的共同努力，所以少年们可以试试帮助开发一下。


[1]:   http://jekyllrb.com/           "jekyll"
[2]:   https://disqus.com/          "Disqus"
[3]:   http://jekyllbootstrap.com/    "Jekyll Bootstrap"
