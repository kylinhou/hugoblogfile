---
title: 将整个hexo博客系统原始文件上传到github
categories: ["博客"]
tags: ["github","博客"]
date: 2016-06-23
author: "kylin"
grammar_cjkRuby: true
---
## 这是个引子
将博客弄得基本差不多了之后忽然发现一个问题，使用hexo deploy命令部署在github上的博客仅仅是hexo系统将执行hexo g之后把md文件转换为静态html之后的文件，（这句话怎么说的这么长~） 那如果某一天电脑崩了，或者是文件丢失了，我们只能从github上弄回这些html文件，我们的这个hexo博客系统是找不回的，菊花一紧！
<!--more-->

这怎么可以，既然有github，我们就要充分利用起来嘛，接下来将整个博客系统的原始文件也弄到github的一个仓库中。

## 几个git命令解析
git init //在当前项目目录中生成本地git管理,并建立一个隐藏.git目录。使用这个命令创建的仓库不是裸仓库，而是在当前目录下生成.git目录，该目录为仓库；而当前目录为工作空间。

git add . //添加当前目录中的所有文件到索引。git add命令主要用于把我们要提交的文件的信息添加到索引库中。当我们使用git commit时，git将依据索引库中的内容来进行文件的提交。

git commit -m "注释" //提交到本地源码库，并附加提交注释。git commit命令，将索引内容添加到仓库中。如果我们这里不用-m参数的话，git将调到一个文本编译器（通常是vim）来让你输入提交的描述信息

git commit -a -m "提交的描述信息"
git commit 命令的-a 选项可只将所有被修改或者已删除的且已经被git管理的文档提交倒仓库中。如果只是修改或者删除了已被Git 管理的文档，是没必要使用git add 命令的。

git remote add origin 仓库地址 //添加到远程项目，别名为origin。git remote是添加一个远程仓库。

git push -u origin master //把本地源码库push到github 别名为origin的远程项目中，确认提交，git push -u origin master第一次推送master分支的所有内容。此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改

## 开始操作
### 为本地hexo文件生成git管理

进入hexo博客的根目录，然后git bash here。在命令行中输入

	git init
这时候将会发现，文件夹中多了一个.git的隐藏文件夹，第一步完成。

### 添加所有文件到索引

继续在命令行中输入

	git add . 
注意这个 .  这样就会将所有的文件加到索引中

### 提交到本地源码库

继续输入

	git commit -m "first commit"
这样就把索引中的所有东西都提交到了本地的源码库中了，提交的注释为：first commit

### 新建远程仓库
到自己的github上，新建一个仓库，仓库的名字随便起，我起了一个hexoBlogFile，拷贝一下这个仓库的地址，下一步使用

### 添加到远程项目
在面板中输入

	git remote add origin https://github.com/xxxx  
上面改成你自己的地址就可以了，这样就把本地的代码块与远程仓库连接起来了

### 将本地源码库push到远程仓库
继续输入

	git push -u origin master
完成本地库push到远程仓库

### 后续的代码更新
以后文件修改了之后，需要不断的更新，只需要如下几个代码就可以了，先进入所在文件夹

	git add .
	git commit -m "update test"
	git push -u origin master
