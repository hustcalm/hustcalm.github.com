---
layout: post
title: "为Octopress博客添加Flag Counter侧边栏"
date: 2012-10-16 11:19
comments: true
categories: [Octopress, SEO] 
---
今天为自己的Octopress博客添加了访问者统计工具，[Flag Counter](http://www.flagcounter.com)，如何获得代码请访问官网。		
拿到了代码后，添加到侧边栏，在custom/asides添加flag_counter.html，然后在_config.yml中开启即可，其中flag_counter.html的格式如下：		

	{% if site.flag_counter %}		
	<section>		
	<h1>Flag Counter</h1>		
	Code you get from [Flag Counter](www.flagcounter.com)		
	</section>		
	{% endif %}	

针对以上代码，还需要在_config.yml文件中添加一行：		

	flag_counter: true	
{% if avoid Liquid Exception %}		
{% endif  %}		

为了在本博文中显示flag_counter.html的第一行和最后一行的开关语句，特意额外添加了一组`if Tag`，Markdown，囧～		

因为，在`rake generate`的时候遇到了Liquid Exceptioon...解决方案参考了[源码](http://code.google.com/p/liquid-markup/source/browse/trunk/lib/liquid/block.rb?r=2)			

**Enjoy your Flag Counter:)**
