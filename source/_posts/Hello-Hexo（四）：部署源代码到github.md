---
title: Hello Hexo（四）：部署源代码到github
date: 2017-07-07 15:54:01
categories: "Hexo教程"
tags: 
      - Hello Hexo
      - hexo
description: 上一篇文章介绍了如何写一篇绘声绘色的博客，本篇介绍如何把你的博客源代码部署到github上以及如何在不同终端编辑博客。
---
# 前言 #
---
现在我们是在一台计算机上编写博客，部署到github的代码是生成的静态化的网页。如果机器瘫痪，就失去了源代码，或者如果想要在另一台机器上继续完成未写完的文章，就需要获取之前的源代码，这就需要把源代码提交到github上。
# 正文 #
---
我们在github上创建仓库时默认只有一个分支：master,而这个分支时存放hexo博客生成的静态页面的，所以只能创建一个分支来存放博客的源代码。
## 创建分支 ##
创建分支有两种方法：
第一种就是在你的github仓库中创建，进入仓库选择仓库页面的Branch按钮：
	{% asset_img pic_01.png pic_01.png%} 
在输入框中填写分支名称回车就保存了分支。
## 更改默认分支 ##
添加完分支后需要修改新的分支为默认分支，点击仓库页面的Settings按钮进入仓库设置页面，设置新分支为默认分支：
	{% asset_img pic_02.png pic_02.png%}
	{% asset_img pic_03.png pic_03.png%}
这样你每次提交的代码就会提交到新的分支中了，而在其他终端clone的代码也会从此分支中下载。
创建分支的第二种方式就是用命令：

	git checkout -b <new branch name> #创建分支并改为默认分支
建议使用第一种更直观些。

关于更多分支的知识可以参考[Git分支管理](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743862006503a1c5bf5a783434581661a3cc2084efa000)
## 配置 ##

>tip：一下命令中都是用的我的github地址，需要替换你自己的地址

在博客根目录的_config.yml配置文件底部中添加配置：

	deploy:
	type: git
	repo: 
		github: git@github.com:rick0407/rick0407.github.io.git,master
## clone项目 ##
clone你的项目到博客根目录下：
	git clone https://github.com/rick0407/rick0407.github.io.git
此命令是把分支的配置信息clone到本地，否则后面提交就会出现冲突。
## 提交代码 ##
执行一下代码将你的源代码提交到github分支中。

	git remote add origin https://github.com/rick0407/rick0407.github.io.git	#关联远程仓库
	git add .	#添加本地所有文件
	git commit -m "your desc"	#提交代码到git版本库中
	git push -u origin branchName	#把当前版本库中的代码提交到远程仓库分支中，其中branchName为默认分支的名称。
如果你已经使用了其他的主题，当你push源代码到github后，会发现在你的分支仓库中新的主题文件夹是灰色的而且打不开，其实是这个主题文件没有被提交到仓库中，这是因为当你clone主题时，也会把主题作者的git环境一同clone下来，这与你自己的git环境不同，就不会被提交到你的仓库中，而主题作者的git环境是在.git文件中，这个文件默认是隐藏的。这个问题也一直困扰我，一直没有找到好的解决办法，介绍一种比较笨但是有效的方法就是把主题文件剪切到其他位置，然后在themes文件夹下创建相同名称的文件夹，把除了隐藏文件.git之外其他文件copy进来，然后再重新提交，这样就会把主题提交到分支中了。

>tip：我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令：

	git push origin branchName

## 在不同终端编写博客 ##
如果在另一台电脑上想继续写未完成的文章，就需要在这台电脑上同样配置nodejs、hexo、git、.ssh。
安装好nodejs后执行：

	git clone https://github.com/rick0407/rick0407.github.io.git
然后在rick0407.github.io文件夹下通过 Git Base依次执行：
	
	npm install hexo
	npm install
	npm install hexo-deployer-git

然后就可以继续编写博客了，编写完成后执行以下命令部署静态网页：

	hexo g
	hexo d
然后执行以下命令把源代码提交到分支中：
	
	git add .
	git commit -m "your desc"
	git push origin branchName

>tip：每次编写博客开始前，需先执行命令:
	
>	git pull origin branchName
>保持本地的版本和分支中的版本一致，然后再编写博客。

# 总结 #
---
本篇介绍了如何提交源代码到分支中以及如何在不同电脑上编写文章。hexo还有许多地方可以根据自己的爱好去修改，但是博客本身的内容比外表更重要，所以还是想办法提高博客的质量。后面的文章会介绍配置第三方服务如：站内搜索等。

# 参考 #
---
[Git远程仓库](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374385852170d9c7adf13c30429b9660d0eb689dd43a000)
[Git分支管理](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743862006503a1c5bf5a783434581661a3cc2084efa000)
