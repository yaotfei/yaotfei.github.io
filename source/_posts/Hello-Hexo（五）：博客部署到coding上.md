---
title: Hello Hexo（五）：博客部署到coding上
date: 2017-07-08 09:36:28
categories: "Hexo教程"
tags: 
      - Hello Hexo
      - hexo
description: 现在我们是把hexo部署到了github上，但是github是国外的网站，每次打开都非常慢，然后看到网上建议可以部署到国内的托管平台，就查询了资料并把代码部署到coding上。
---
# 前言 #
---
coding是和github一样的国内的代码托管平台，部署方式和github类似。
# 正文 #
---
## coding创建新项目 ##
想要把代码部署到coding上当然是要在[coding官网](https://coding.net)注册账号。
注册完后同样需要创建项目，相当于github的仓库：
	{% asset_img pic_01.png pic_01.png%}
项目建好后复制项目的ssh地址，或者https地址
	{% asset_img pic_02.png pic_02.png%}
然后在博客的_config.yml最下面配置：

	deploy:
	type: git
	repo: 
		github: git@github.com:rick0407rick0407.github.io.git,master
		coding: https://git.coding.net/rick0407/rick0407.git,master

>tip：_config.yml文件是YAML标记语言，YAML依靠缩进来确定元素间的从属关系。因此，请确保每个deployer的缩进长度相同，并且使用空格缩进。


把之前本地生成的.ssh公钥（id_rsa.pub文件中）配置到coding中
	{% asset_img pic_03.png pic_03.png%}
添加完后，准备工作就完成了，
在git bash输入命令：
	
	ssh -T git@git.coding.net
遇到提示输入yes：
	{% asset_img pic_04.png pic_04.png%}
出现如上结果说明公钥添加成功。
最后把博客同步到coding中：

	hexo deploy -g
同步后怎么才能像github那样访问自己的博客呢。
coding访问自己项目的方式是

	http://登陆名.coding.me/项目名
比如我的是 

	http://rick0407.coding.me/rick0407
如果你的项目名和登录名是一样的就可以省略项目名，直接输入 

	http://rick0407.coding.me
也可以访问，所以建议项目名和登录名相同。
同步到coding后还不能直接访问，需要在Pages服务里面保存部署来源：
	{% asset_img pic_05.png pic_05.png%}
这样就可以直接访问你的博客了。
## 绑定域名 ##
把你的github地址和coding地址绑定到同一个域名下，实现国内的访问coding，国外访问github。
在[阿里云](https://wanwang.aliyun.com)购买域名，（当然也可以在其他域名提供商购买），我们自己用就没必要购买.com的国际域名和白金词，最便宜的3元/年。购买后需要实名制和简单的配置。
通过审核后就可以配置域名了，在博客根目录的source目录下创建CNAME文件（文件名称必须一样），没有后缀。文件的内容是你自己的域名，不带www,如yaotfly.com。添加后部署到github上和coding上面。
部署到上面后，需要在coding网站上绑定自己的域名
	{% asset_img pic_06.png pic_06.png%}
绑定后，在域名控制台对自己的域名添加解析：
	{% asset_img pic_07.png pic_07.png%}
然后就可以访问自己的域名了，当你访问原部署的地址时，会自动跳到自己的域名下。

# 总结 #
---
到此，我们终于实现了部署一次github和coding同步部署了，访问速度也比之前快了很多，后面将为我们的博客添加站内搜索，让百度能狗收录我的博客

# 参考 #
---
[hexo博客同时部署至github和Coding](http://blog.csdn.net/u011303443/article/details/51509351)
[hexo博客添加域名实现双线部署（github和coding）](http://blog.csdn.net/qiuchengjia/article/details/52923156)
