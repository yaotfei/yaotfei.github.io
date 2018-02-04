---
title: Hello Hexo（六）：为博客添加站内搜索
date: 2017-07-09 17:57:57
categories: "Hexo教程"
tags: 
      - Hello Hexo
      - hexo
description: 每个主题都有预留了搜索功能，只是需要我们自己完成配置工作，而且每个主题的搜索功能配置有所不同，本篇以我自己是用的主题为例。
---
# 概述 #
---
我们可以是用第三方服务來作为自己网站的站内搜索，开始想用百度的站内搜索，但是没成功，查询资料得知是因为百度站内搜索不支持https协议，所以只有换swiftype，这是国外的服务商，最好翻墙。
# 正文 #
---
## 注册 ##
在[swiftype官网](https://app.swiftype.com/)注册账号，提示免费是用7天，不过网上说过了这个期限也可以用，只不过功能少了，对于我们够用了。
## 创建搜索引擎 ##
注册登录后，点击create an engine--输入你的网址--给搜索引擎起名字：
	{% asset_img pic_01.png pic_01.png%}
	{% asset_img pic_02.png pic_02.png%}
	{% asset_img pic_03.png pic_03.png%}
创建好搜索后，过几分钟就可以看到从网站抓取的数据了：
	{% asset_img pic_04.png pic_04.png%}
## 安装 ##
配置成功后接下里就是在你的网站安装搜索功能了。
点击左下的`install Search`显示搜索插件的代码，
	{% asset_img pic_05.png pic_05.png%}
用户可以根据自己需求是用swiftype提供的样式还是自定义，如果网站有定义好的搜索框样式可以点击`Change Configuration`按钮自定义自己的样式。
	{% asset_img pic_06.png pic_06.png%}
修改完后 点击`Save Changes`保存修改，点击`Activate Swiftype`激活。激活后复制插件的代码。
## 主题配置 ##
接下来就是配置主题了：
首先把主题配置文件_config.yml中的搜索属性设置为true，我用的主题设置是：
	
	# 是否显示边栏中的搜索框
	# Search Box in left column
	search_box: true
然后找到自己主题中关于搜索ejs文件，我的是themes\spfk\layout\_partial\left-col.ejs
	{% asset_img pic_07.png pic_07.png%}
把在swiftype中设置的搜索框的代码copy进去。
然后就是把搜索插件的代码放入网页中，swiftype上说把代码放到所有网站的网页中，我们可以放在一个公共的网页，我放的是themes\spfk\layout\_partial\after-footer.ejs文件底部。
	{% asset_img pic_08.png pic_08.png%}
然后就是创建搜索结果页面，在themes\spfk\layout中创建search.ejs文件。文件内容是：


 	<% if(theme.search_box) { %>
	<div  id="container" class="page">
	  <div id="st-results-container" class = "st-default-search-input" style="width:80%">正在加载搜索结果，请稍等。</div>
	  <style>.st-result-text {
	  background: #fafafa;
	  display: block;
	  border-left: 0.5em solid #ccc;
	  -webkit-transition: border-left 0.45s;
	  -moz-transition: border-left 0.45s;
	  -o-transition: border-left 0.45s;
	  -ms-transition: border-left 0.45s;
	  transition: border-left 0.45s;
	  padding: 0.5em;
	}
	@media only screen and (min-width: 768px) {
	  .st-result-text {
	    padding: 1em;
	  }
	}
	.st-result-text:hover {
	  border-left: 0.5em solid #ea6753;
	}
	.st-result-text h3 a{
	  color: #2ca6cb;
	  line-height: 1.5;
	  font-size: 22px;
	}
	.st-snippet em {
	  font-weight: bold;
	  color: #ea6753;
	}</style>
	
	<!--注意下面到</script>结束的代码块要替换成自己上面保存的Install Code代码--!>
	<script type="text/javascript">
	  	  (function(w,d,t,u,n,s,e){w['SwiftypeObject']=n;w[n]=w[n]||function(){
	  (w[n].q=w[n].q||[]).push(arguments);};s=d.createElement(t);
	  e=d.getElementsByTagName(t)[0];s.async=1;s.src=u;e.parentNode.insertBefore(s,e);
	  })(window,document,'script','//s.swiftypecdn.com/install/v2/st.js','_st');
	  
	  _st('install','6Rkyy_E5GJpZAdsHY_4k','2.0.0');
	</script>
	<% } %>
现在搜索功能就已经完成了：
	{% asset_img pic_09.png pic_09.png%}

# 总结 #
---
第三方的站内搜索很多，如果有空，大家也可以试试其他的搜索。

# 参考 #
---
[利用swiftype为hexo添加站内搜索v2.0](http://www.jerryfu.net/post/search-engine-for-hexo-with-swiftype-v2.html)
