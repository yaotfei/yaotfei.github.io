---
title: Hello Hexo（二）：hexo主题及网站基本配置
date: 2017-07-07 09:24:04
categories: "Hexo教程"
tags: 
      - Hello Hexo
      - hexo
description: 上一篇介绍如何用github+hexo搭建自己的播客，hexo默认的主题是landsacpe，如果不喜欢也可以从github上面选择自己喜欢的主题。本篇介绍一下我自己用的主题配置
---
# 概述 #
---
上一篇介绍如何用hexo+hexo搭建自己的播客，hexo默认的主题是landsacpe，如果不喜欢也可以从github上面选择自己喜欢的主题。本篇介绍一下我自己用的主题配置
# 正文 #
---
## 安装主题 ##
进入[hexo主题](https://hexo.io/themes/)网页选择自己喜欢的主题，这里面列出的都是其他网友制作的主题，点击主题图片可以预览，喜欢哪个就点击主题名称进入github复制网址输入指令：

	git clone https://github.com/luuman/hexo-theme-spfk themes/spfk
这样主题spfk就下载到本地了，位于博客网站根目录下的themes文件夹内
## 更换主题 ##
修改博客根目录下的_config.yml文件中的theme属性为spfk
修改后启动

	hexo server -g #生成加预览
这样就成功更换了主题
## 修改配置信息 ##
主题更换后，里面的部分内容如博客名称目录显示的是主题作者的信息，需要改成自己的信息，每一个作者做出的主题的配置项是不一样，但基本的配置是大致相同的，这里只介绍基本的配置，其他内容就需要自己探索了。
在主题被下载到本地后，目录结构为

>1. |主题目录结构
1.  |----languages
1.  |----layout
2.  |----source
3.  |----.gitignore
4.  |----_config.yml
5.  |----README.md
 
languages为主题所支持的语言
layout为布局模版
source存放主题的资源文件
.gitignore的作用是忽略提交的文件
_config>yml为主题的配置信息
README.md为主题作者的介绍

主题的配置信息在主题根目录下的_config.yml文件中，不建议使用系统自带记事本编辑，可能会出现中文乱码，如果使用notepad++进行编辑，要把编码改成utf-8编码。
每一个主题的配置文件作者都做了大致的注释（也有可能是英文注释）

	menu:
 	博客: /Home
 	所有文章: / #指向博客首页
 	标签: /tags
 munu显示的是博客的目录，如果想添加自己的目录就按上面的格式添加，然后在博客根目录下的source文件夹内添加相应的文件夹。
如：若想添加标签目录，在menu属性下添加
	标签： /tags
然后在source中创建tags文件夹，在tags文件夹中创建index.md文件，添加内容：

	layout: tags
	title:tags
	---

这样在博客的目录中就添加了tags目录，只不过里面没有内容。
如果想把博客的头像或者博客的背景改成自己喜欢的，就需要修改source文件中的内容了，博客中的图片、样式，js等都放在这里面，可以根据需要进行修改
在主题的配置文件中还可以配置博客的作者名称、社交链接以及整合站内搜索、统计等第三方功能，这个在后面内容介绍。
在博客根目录下的_config.yml配置文件中可以配置站点的信息
具体配置可以参考[hexo配置信息](https://hexo.io/zh-cn/docs/configuration.html)

# 总结 #
---
本文简要介绍了主题的配置和网站的基本信息配置，详细的配置需要参考hexo官网的介绍以及主题作者的注释信息。
下一篇将介绍如何新建一篇博客。

# 参考 #
---

[hexo主题列表](https://hexo.io/themes/)
[hexo配置指南](https://hexo.io/zh-cn/docs/configuration.html)
