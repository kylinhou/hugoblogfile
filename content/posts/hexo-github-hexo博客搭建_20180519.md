---
title: github + hexo 博客搭建 
categories: ["博客"]
tags: ["博客搭建", "github", "hexo"]
grammar_cjkRuby: true
date: 2016-06-30 
---


## git安装
git的相关教程非常多，这里就不赘述了
<!-- more -->
## node.js安装
去官网下载 http://nodejs.org/download/ ，之后安装，我使用了默认安装路径

## 安装Hexo
在电脑的屏幕上右键（随便选择桌面上就行），选择Git Bash Here ，然后出现终端界面，输入npm install -g hexo
回车之后等待，但是出错，报错如下

> npm WARN optional Skipping failed optional dependency /hexo/chokidar/fsevents:
> npm WARN notsup Not compatible with your operating system or architecture: fsevents@1.0.12
> npm ERR! Windows_NT 10.0.10586
> npm ERR! argv "C:\\Program Files\\nodejs\\node.exe" "C:\\Program Files\\nodejs\\node_modules\\npm\\bin\\npm-cli.js" "install" "-g" "hexo"
> npm ERR! node v6.2.2
> npm ERR! npm  v3.9.5
> npm ERR! shasum check failed for C:\Users\kylin\AppData\Local\Temp\npm-12620-bfae7f80\registry.npmjs.org\JSONStream\-\JSONStream-1.1.3.tgz
> npm ERR! Expected: 1b296c7949b47e4fc22ac7958f76d2fbe2415493
> npm ERR! Actual:   46eb6043a766a9b5f25d516cd21373227d912302
> npm ERR! From:     https://registry.npmjs.org/JSONStream/-/JSONStream-1.1.3.tgz
> npm ERR!
> npm ERR! If you need help, you may report this error at:
> npm ERR!     <https://github.com/npm/npm/issues>
> npm ERR! Please include the following file with any support request:
> npm ERR!     C:\Users\kylin\Desktop\npm-debug.log

出错的原因不明白，尝试各种解决方法：
* 将安装好的最新版卸载掉，重新下载了稳定版v4.4.4安装重试
* 以管理员身份运行Git Bash，然后再输入npm install -g hexo，成功！！！

创建一个文件夹，保存自己的hexo网站，在文件夹内右键，选择Git Bash Here ，输入

			 hexo init
			 npm install

输入hexo server，出现下图的话表示hexo运行成功

![hexo运行成功][1]

浏览器访问 http://localhost:4000 ，看一下会有什么效果

## 开始Github
1.    github账号以前已经注册过，直接使用
2.    建立一个仓库，与用户名对应的仓库，仓库名必须为『your_user_name.github.io』
3.    将本地的ssh-key复制到github上

              ssh-keygen -t rsa -C "你的github注册邮箱"
       然后一直回车，直到出现

       ![enter description here][2]
       查看电脑中以前的ssh-key，输入：
      ​    
           cd ~/.ssh/ && ls
       如果结果为： id_rsa  id_rsa.pub继续进行，输入：
      ​    
      	cat id_rsa.pub
       将会显示出SSH-KEY，复制内容粘贴到github的ssh中
4.    在hexo目录中有一个  config.yml 文件, 在其最后补全如下信息：
      deploy:
      type: git
      repo: https://github.com/yourname/yourname.github.io.git
       branch: master
       执行

            hexo deploy

      出错：
      > ERROR Deployer not found: git

    找到解决方案：
    需要先安装hexo-deployer-git，然后将github修改为git
    
          npm install hexo-deployer-git --save
    
    之后重新执行hexo deploy
    弹出一个对话框，输入github账户名和密码，完成之后，成功将本地的hexo文件上传到github

在使用ssh连接github的时候一直不成功，输入ssh -T git@github.com显示：

          $ ssh -T git@github.com
          The authenticity of host 'github.com (192.30.253.112)' can't be established.
          RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
          Are you sure you want to continue connecting (yes/no)?
          Host key verification failed.

尝试了很多次依然是不管用，发现这个问题有点白痴，要在Are you sure you want to continue connecting (yes/no)?这句话后面填上yes，回车才能成功！！！！

## 使用GitHub Pages建立博客
在搜索hexo的相关资料的时候发现了next主题，所以首先使用next主题看一下效果，next的地址为 http://theme-next.iissnan.com/ 
上面有详细的配置说明，可以把主题设置的更加适合自己。


#### 发表新文章
* hexo new "文章标题"（可以简写new为n）
* 我喜欢直接在source\_posts文件夹下新建一个.md文件，效果是一样的，或者把在其他地方写好的md文件直接拷贝过来

然后继续 hexo g，hexo d，文章就部署好了

#### 删除文章
删除文章非常简单，直接把相应的.md文件在删除，然后执行hexo g，hexo d就OK了

#### 首页显示文章摘要
在next主题使用说明中有说明，一共三种方法：

* 在文章中使用 <!-- more --> 手动进行截断，Hexo 提供的方式 **推荐**
* 在文章的 front-matter 中添加 description，并提供文章摘录
* 自动形成摘要，在 主题配置文件 中添加：

      auto_excerpt:
        enable: true
        length: 150
  默认截取的长度为 150 字符，可以根据需要自行设定

建议使用 <!-- more -->（即第一种方式），除了可以精确控制需要显示的摘录内容以外， 这种方式也可以让 Hexo 中的插件更好的识别。

#### 添加独立域名
很早之前就买了一个域名，kylin10.com,但是确一直没有用起来，之前的博客网站放在新浪云SAE上，域名没有备案无法绑定，SAE的费用也是越来越高，用完了云豆就来折腾github了。

将github与自己的域名绑定只需下面几部：

1. 在Hexo的source目录下新建一个CNAME文件，里面写上自己的域名，注意前面不要加www等内容，直接写kylin10.com，完成之后deploy
2. 注册[DNSpod](https://www.dnspod.cn)，添加域名,可以按照下图设置：
   ![图片描述](http://cnfeat.qiniudn.com/15.png)
   图片来自http://www.jianshu.com/p/05289a4bc8b2/

3.在购买域名的管理中心修改域名对应的dns

#### 同时部署至github和Coding
同时将博客部署到github跟coding上，可以参考这个教程 http://sanwen8.cn/p/1a0TJhT.html

但是有一点需要注意：

在给coding上添加ssh的时候要选择个人公钥！！！，默认的是公开公钥，第一次我没有注意，添加之后git一直出错。

#### 站内搜索
不知道为什么，站内搜索一直失败，还在努力中


