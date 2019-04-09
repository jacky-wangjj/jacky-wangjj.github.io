---
layout: post
title: GitHub+Jekyll搭建个人博客详解
date: 2019-04-09 
tags: 环境
---
### 工具
* github
* jekyll
* 百度统计
* 来比力评论系统
* 不蒜子

# 搭建过程

### 网站托管

直接使用GitHub提供的GitHub Pages功能，展示博客的页面。
    
1. 首先需要到[GitHub](https://github.com/)上注册账号，例如我注册的是jacky-wangjj；
    
2. 点击New repository -> 输入仓库名称（格式为<用户名>.github.io，如jacky-wangjj.github.io）-> 点击Create repository；

### Jekyll安装

这里讲解的是Windows下的安装步骤。

1. 首先点击下载安装[Ruby installer](https://rubyinstaller.org/downloads/)，我这里使用的是Ruby 2.3.3；

2. 然后下载[DevKit](https://rubyinstaller.org/downloads/)，我这里使用的是DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe；

3. 检查Ruby installer是否安装完成，win+R打开命令行，输入ruby -v，出现版本信息说明安装成功；

4. 将DevKit解压到指定目录，如：D:\DevKit。然后命令行进入该目录cd D:\DevKit，执行：
    ruby dk.rb init
    ruby dk.rb install *
     
5. 安装jekyll，输入如下命令：
    gem sources --remove https://rubygems.org/
    gem sources -a http://gems.ruby-china.com/
    gem sources -l
    gem install jekyll
    
6. 测试jekyll环境
    新建一个blog：jekyll new blog
    启动服务：进入目录输入jekyll serve
    查看页面：浏览器访问[localhost:4000](http://localhost:4000)
    
### 下载博客模板并修改

我这里直接使用的[leopardpan](https://github.com/leopardpan/leopardpan.github.io)修改过的模板，按照自己需要进行修改即可。

博客目录结构如下：
```
.
├── _config.yml         #配置信息
├── _includes           #部分子模块
├── _layouts            #自定义html显示
├── _posts              #博客上传目录
├── _site               #jekyll生成的HTML文件
├── 404.html
├── about.md            #关于我
├── archive.html
├── css                 #css样式
├── feed.xml            #RSS订阅相关
├── rss.xsl             #RSS订阅相关
├── images              #图片资源
├── index.html
├── js                  #js文件
├── README.md           #项目介绍
└── tags.html

```
1. 修改_config.yml文件；
2. 添加[百度统计](https://tongji.baidu.com/web/welcome/products?castk=LTE%3D)；
    注册百度统计账号，新增网站 -> 输入网站域名：jacky-wangjj.github.io；网站首页：https://jacky-wangjj.github.io -> 确定；
3. 添加[来比力评论](https://www.livere.com/)；
    注册来比力账号，关联网站的URL；
4. 添加[RSS订阅](https://www.jianshu.com/p/da39860bb5f5)；
5. 使用[不蒜子](https://www.cnblogs.com/daoyi/p/jing-tai-wang-zhan-shi-yong-bu-suan-zi-xian-shi-li.html)统计浏览量；

### 提交代码到GitHub

将修改完成的代码提交到github仓库；
添加博客时可将编写好的markdown文件放入_posts目录，以特定格式命名：2019-04-09-blog's name.md；
然后提交至GitHub；

### 编辑markdown文件

我这里使用的是[IDEA + MultiMarkDown插件](https://www.jianshu.com/p/a0550f81cbd1)，不用装太多其他软件就可以编辑MarkDown文件；