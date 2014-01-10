---
layout: post
title: "git的global config和repo config"
date: 2012-10-23 18:26
comments: true
categories: Git 
---
相比于CVS，SVN，现在越来越喜欢Git了，下面说说Git的全局配置，即global config以及代码库级别的配置，即repo config。

## Global Config
git的全局配置，存储在$HOME/.gitconfig这个文件里面，对当前用户的所有git repo有效。

### 基本配置
    git config --global user.name yourname
    git config --global user.email youremail

### 默认编辑器
    git config --global core.editor vim

### 默认merge/diff工具
设置后需要通过difftool和mergetool启动。
    git config --global diff.tool vimdiff
    git config --global difftool.prompt false

### Alias设置别名
通过别名设置，提高操作效率。        

<!--more-->

分支切换
    git config --global alias.br branch
    git br
查看日志
    git config --global alias.last log --pretty=oneline -1 HEAD
    git last
设置difftool
    git config --global alias.df difftool
    git df
提交代码
    git config --global alias.cm commit
查看版本库状态
    git config --global alias.st status
查看远程版本库
    git config --global alias.re remote

## Repo Config
repo级别的配置，存储在仓库目录的.git/config文件中，可以覆盖全局信息。如果用不同的用户名和邮箱参加不同的项目，这里设置即可，还包括
不同项目git使用习惯的一些不同。
配置命令如下：
    git config user.name yournewname
    git config user.email yournewemail
git本身也会存储远程仓库以及分支列表等信息：
    cat .git/config

## References

*   [Frank Xu](http://f2e.us/wiki/git-config.html#!/)
*   [git-config](http://www.kernel.org/pub/software/scm/git/docs/git-config.html)

