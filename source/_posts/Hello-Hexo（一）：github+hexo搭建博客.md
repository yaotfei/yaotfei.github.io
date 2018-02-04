---
title: Hello Hexo（一）：github+hexo搭建博客

date: 2017-07-04 16:00:12 

categories: "Hexo教程"

tags: 
      - Hello Hexo
      - hexo
description: 作为一名不入流程序猿，遇到问题都是找其他人的博客，渐渐发现他们的博客都是相互借鉴的，第一个方法不合适，后面的都不用看了，所以自己也想做一个博客来记录自己遇到的问题，但是作为不喜欢随波逐流的人，当然不喜欢在第三方博客上写了，所以很早就萌生了自己搭建博客的想法，但一直没有动手做，因为不会。。。最近才听说hexo这个框架，平时工作忙，业余时间折腾了一周，终于出成果了，老泪纵横啊，然后赶快记录下自己的虐心过程，以此来开始我的博客生涯。
---
# 概述 #
---
作为一名不入流程序猿，遇到问题都是找其他人的博客，渐渐发现他们的博客都是相互借鉴的，第一个方法不合适，后面的都不用看了，所以自己也想做一个博客来记录自己遇到的问题，但是作为不喜欢随波逐流的人，当然不喜欢在第三方博客上写了，所以很早就萌生了自己搭建博客的想法，但一直没有动手做，因为不会。。。最近才听说hexo这个框架，平时工作忙，业余时间折腾了一周，终于出成果了，老泪纵横啊，然后赶快记录下自己的虐心过程，以此来开始我的博客生涯。
# 正文 #
---

我的软件环境是window系统用户，Hexo3版本
## 准备工作 ##
闲言碎语不多说，直接入正题
hexo是运行再node.js环境下的，所以要下载安装[Node.Js](http://nodejs.cn/download/ "Node.Js"),安装过程略过。
我们要把自己的博客托管到github上，所以要在[github官网](https://github.com/ "github官网")注册自己的账号.
下载[git](https://git-scm.com/ "git")用来管理自己的仓库

更详细的相关教程可以参考<a href="#end">文末</a>的网站
## 安装hexo ##
安装好node.js和Git后就可以安装hexo了
安装好node.js后，任意位置右键选择Git Bash Here选项
安装好后会用到以下命令，#为注释：

	hexo g #用于生成静态文件
	hexo s #用于启动服务器，可用来本地预览
	hexo d #用于将本地文件发布到github上
	hexo n #用于新建一片文章
输入安装hexo命令：

    npm install -g hexo
安装完成后，建一个文件夹（如：D:\HexoBlog），执行以下命令，Hexo自动在文件夹中建立网站所需所有文件：

    hexo init
安装依赖包：

    npm install
然后在D:\HexoBlog文件夹下执行以下命令：

    hexo g
	hexo s
然后用浏览器访问http://localhost:4000, 就可以看到部署在本地的博客了。
## 部署hexo到github ##
以上只是把自己的博客部署到了本地，如果想让别人看到你的成果，那就需要把博客部署到外部服务器上，或者部署到github上。
## 注册github ##
想要把代码部署到github上，当然要注册[github](https://github.com/ "github")账号，记住登录名，后面要用到。
## 创建仓库 ##
仓库用来存储代码，注册账号登录后创建一个仓库
{% asset_img pic_01.png pic_01.png%}
创建时，填写Repository name,仓库的名字格式为：yourname.github.io,其中yourname为注册github时的登录名。
{% asset_img pic_02.png pic_02.png%}
## 本地文件部署到github ##
想要把代码部署到github上，必须修改网站文件中的配置文件:_config.yml,建议使用Notepad++编辑。
在_config.yml文件最下，添加如下配置：

    deploy:
      type: git
      repo: 
      	 github: git@github.com:rick0407/rick0407.github.io.git,master
注意：hexo的配置文件中任何冒号后面都是带一个空格的，
github的值是你仓库的cloneURL。
配置完_config.yml后，如果是第一次使用github还需要配置SSH。
在Git Bash Here中输入一下指令可以查看是否存在.ssh文件，如果有可以删除。

	$ ls -al ~/.ssh
	total 34
	drwxr-xr-x 1 RICK 197121    0 六月 27 17:16 ./
	drwxr-xr-x 1 RICK 197121    0 七月  6 09:08 ../
	-rw-r--r-- 1 RICK 197121 1679 六月 27 16:36 id_rsa
	-rw-r--r-- 1 RICK 197121  398 六月 27 16:36 id_rsa.pub
	-rw-r--r-- 1 RICK 197121  407 六月 27 17:16 known_hosts
输入一下指令后，提示输入用户名密码时不用填写一路回车：

	ssh-keygen -t rsa -C "注册github时的邮箱"
然后输入一下指令

	ssh-agent -s
继续输入

	ssh-add ~/.ssh/id_rsa
输入后如果出现could not open a connection to your authentication agent错误就先输入

	ssh-agent bash
然后继续执行上一条指令
成功后就可以添加SSH key到你的github账户了。
复制本地id_rsa.pub中的内容（C:\Users\administrator\.ssh\id_rsa.pub）到github中
{% asset_img pic_03.png pic_03.png%}
{% asset_img pic_04.png pic_04.png%}
复制后可以测试一下，输入一下指令

	ssh -T git@github.com
如果有警告输入"yes"就行
{% asset_img pic_05.png pic_05.png%}
最后要安装hexo-deployer-git模块

	npm install hexo-deployer-git --save
安装好后执行以下命令

	hexo g	#生成静态文件
	hexo d	#部署
部署后在地址栏输入yourname.github.io就可以访问了，比如我的是rick0407.github.io

# 总结 #
---
到此就成功创建了一个hexo博客网站，这只是最基本的流程，hexo支持许多插件，和好看的主题，可以让你的博客页面更好看，后面会增加一些插件，更换主题。

# 参考 #
---
<a id="end"></a>

 [hexo中文网](https://hexo.io/zh-cn/ "hexo中文网")
 [nodejs中文网](http://nodejs.cn/ "nodejs中文网")
 [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000 "Git教程")

