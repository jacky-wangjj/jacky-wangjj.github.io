---
layout: post
title: 【已解决】OpenSSL SSL_connect: Connection was reset in connection to github.com:443
date: 2022-02-16
tags: github
---  
# 【已解决】OpenSSL SSL_connect: Connection was reset in connection to github.com:443

今天在使用git命令进行push和pull时，出现如下报错     
```bash
Git Pull Failed: unable to access 'https://github.com/jacky-wangjj/jacky-wangjj.github.io.git/': OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
```

## 方案一

在git bash命令行中依次输入以下命令：
```bash
git config --global http.sslBackend "openssl"
git config --global http.sslCAInfo "D:\JavaEE\DevelopTools\Git\mingw64\ssl\cert.pem"
```
注意上面第二个命令，路径要换成git安装的路径。

## 方案二

如果你开启了VPN，很可能是因为代理的问题，这时候设置一下http.proxy就可以了。

一定要查看自己的VPN端口号，假如你的端口号是7890，在git bash命令行中输入以下命令即可：
```bash
git config --global http.proxy 127.0.0.1:7890
git config --global https.proxy 127.0.0.1:7890
```

如果你之前git中已经设置过上述配置，则使用如下命令取消再进行配置即可：
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

下面是几个常用的git配置查看命令：
```bash
git config --global http.proxy #查看git的http代理配置
git config --global https.proxy #查看git的https代理配置
git config --global -l #查看git的所有配置
```

亲试方案一有效