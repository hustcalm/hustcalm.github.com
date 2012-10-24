---
layout: post
title: "使用git和github来管理vim配置和插件"
date: 2012-10-20 16:29
comments: true
categories: VIM 
---
现在越来越觉得自己离不开Linux了，也越来越离不开VIM了，虽然一直有想学习下Emacs的心思，不过学习曲线太陡峭了，准备细水常流了～之前在windows下使用cygwin，配置VIM用的是`.vim_runtime`，作者是用svn管理的。今天再去看，发现作者也把自己的.vimrc以及整个配置都迁移到了github上，因此参考了[这篇文章](http://blog.pkufranky.com/2011/11/使用git和github来管理vim配置和插件)。请作者原谅我使用了一样的题目～	
文章的目标为：

*	将vim的配置文件使用git管理，并托管到github上

*	通过github来安装和更新vim plugin
<!--more-->
## 使用git管理vim配置文件并托管到github上 ##
### 安装git ###
	sudo apt-get install git-core
### 安装git-subtree ###
	cd
	mkdir github && cd github
	git clone git://github.com/apenwarr/git-subtree.git
	cd git-subtree
	sudo sh install.sh
### 使用git管理.vim目录 ###
	cd
	mkdir .vim
	git init
	touch README
	git add .
	git commit -a -m 'Initial commit of .vim configuration'
	touch vimrc
	git add .
	git commit -a -m 'add vimrc'
	git remote add 'your repo at github that you created for vim conf'
	git push
### 使vimrc配置生效 ###
	ln -s .vim/vimrc .vimrc
## 从github安装和更新vim plugin ##
### bootstrap：安装vim-pathogen ###
vim-pathogen可以使得每个vim plugin单独放到bundle目录下，方便管理和更新。	

首次安装：	
	cd 
	cd .vim && mkdir bundle
	git subtree add \
		--prefix=bundle/vim-pathogen \
		--squash \
		https://github.com/tpope/vim-pathogen.git master
更新：
	git subtree pull \
		--prefix=bundle/vim-pathogen \
		--squash \
		https://github.com/tpope/vim-pathogen.git master
配置：
	runtime bundle/vim-pathogen/autoload/pathogen.vim
	call pathogen#infect()
### 从github安装其它plugin ###
过程类似安装vim-pathogen，也可以使用[pkufranky在github的脚本](https://github.com/pkufranky/vimconf/blob/master/github-plugin-install.sh)来简化安装。	
比如安装或更新 `https://github.com/hallison/vim-markdow`
	sh github-plugin-install.sh hallison/vim-markdown
脚本会根据bundle下对应目录是否存在，决定是否安装或者跟新。	
Done:)
