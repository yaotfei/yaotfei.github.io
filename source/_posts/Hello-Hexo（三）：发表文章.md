---
title: Hello Hexo（三）：发表文章
date: 2017-07-07 11:27:41
categories: "Hexo教程"
tags: 
      - Hello Hexo
      - hexo
description: 博客框架已经搭建好了，配置文件也配置了，接下来就是写自己的博客了，本文介绍如何写自己的播客以及所用的语法。
---
# 概述 #
---
博客框架已经搭建好了，配置文件也配置了，接下来就是写自己的博客了，本文介绍如何写自己的播客以及所用的语法。
<!--more-->
# 正文 #
---
## 博客目录介绍 ##
在博客网站新建完后根目录的文件结构如下：

>1.     .
1.     |----_config.yml
1.     |----package.json
1.     |----scaffolds
1.     |----source
1.     |&nbsp;&nbsp;&nbsp;&nbsp; |----_drafts
1.     |&nbsp;&nbsp;&nbsp;&nbsp; |----_posts
1.     |----themes

scaffolds为模版文件夹。当新建文章时，Hexo会根据sacffolds来建立文件。
Hexo的模板是指在新建的markdown文件中默认填充的内容。例如，如果您修改scaffold/post.md中的Front-matter内容，那么每次新建一篇文章时都会包含这个修改。
source为资源文件夹，是存放用户资源的地方。除了_posts文件夹外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去
## 新建一篇文章 ##
通过在shell中执行以下命令来创建一篇新文章

	hexo new [layout] <title>
layout为文章指定布局，默认为post，执行命令后，hexo在source/_post中创建了.md文件，文件名称就是文章的title。
hexo使用的是markdown标记语言，所以写文章时要遵循markdown语法。你可以用notepad++编辑，也可以用其他
如markdownpad编辑器。
打开.md文件后里面已根据模版添加了一些内容：

	 ---
	 title: 文章标题
	 date: 2017-07-07 11:27:41
	 tags:
	 ---
title为文章标题
date为发表时间，用户可以再配置文件中修改时间格式
tags为文章的标签
用户可以添加categories属性为文章添加类别，方便以后查找。
详细的配置可以参考[hexo官网文档](https://hexo.io/zh-cn/docs/writing.html)。
## 为文章目录添加摘要 ##
我们刚建好的博客框架初始化了一篇文章：hello world，但是默认是显示全文的，如果之后文章数增多，页面看这比较乱，所以，可以设置文章目录显示摘要，或者只显示title，若想显示摘要，需在主题的配置文件中配置显示摘要的属性（具体配置每个主题不相同，可查看注释），然后在写博客时指定摘要的内容，具体有两种方法：
一：在文章正文中添加

	<!--more-->
则之前的文字就会作为摘要显示在目录下方。
二：就是在文章头加上description属性，description的值就作为文章摘要显示在目录下方。

	 ---
	 title: 文章标题
	 date: 2017-07-07 11:27:41
	 tags:
	 description： 文章摘要
	 ---

## 为文章添加图片 ##
想要在文章中插入图片可以按照markdown语法来插入格式为

	![图片名称](图片地址)
图片的存放有两种方式：在本地D:\Hexo\source`目录下新建一个存放图片的文件夹，比如images`，然后把想要插入的图片放在里面，插入图片的路径；第二种方法是利用第三方网站（如：[图床](http://tuchuang.org/)、[七牛云存储](https://portal.qiniu.com)）把图片上传到网络，然后插入图片路径。
hexo3.x还提供了另一种解决方案：
安装 post_asset_folder

	npm install https://github.com/CodeFalling/hexo-asset-image
把博客根目录中配置文件的属性post_asset_folder设置为true,
这样在你每次新建一篇文章时，在source/_posts目录中都会增加一个同名的文件夹，利用一下语法来引用图片：

	{% asset_link slug [title] %}
如：

	{% asset_img example.jpg This is an example image %}

## 插入音乐 ##
我们还可以利用第三方音乐生成外链放入文章中添加音乐：

	<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=432506345&auto=1&height=66"></iframe>
然后页面上就会出现播放器：
><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=432506345&auto=0&height=66"></iframe>

>tip:网易云音乐外链中auto=1时会自动播放，设置为0时为手动播放。我们也可以手动修改宽高等属性。

同理，我们也可以插入视频播放器等让自己的文章绘声绘色。

# 总结 #
---
这片文章简要介绍了新建文章以及编辑文章的方法，具体markdown语法可参考以上网站。我们写完博客后需要部署到github上可以再外网上浏览，但是github上存放的只是生成后的静态页面而不是源代码，如果我们编写博客的电脑发生故障就会丢失源代码，这是很危险的，下一篇就介绍如果把源代码提交到github以及如何在不同计算机上写博客。

# 参考 #
---
 [markdown语法](http://www.appinn.com/markdown/)
 [hexo基本操作](https://hexo.io/zh-cn/docs/writing.html)

