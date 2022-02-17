---
layout: post
title: 【已解决】Logon failed, use ctrl+c to cancel basic credential prompt
date: 2022-02-16
tags: github
---  
# 【已解决】Logon failed, use ctrl+c to cancel basic credential prompt.

本地推送代码带Git仓库失败,报错Logon failed, use ctrl+c to cancel basic credential prompt.

推送的时候弹出githup的登陆框,账号密码正确但是提示不正确

解决方法:在网页上登陆你自己的githup账号,点击右上角头像-->  setting --> Developer settings --> Personal access tokens页面

![](https://jacky-wangjj.github.io/images/blog/env/github/qunar-push.png#pic_center)

点击新建 genrate new token

![](https://jacky-wangjj.github.io/images/blog/env/github/push-token.png#pic_center)

新建完成,页面已经有一个新的token,这个页面先不要动,或者先复制出来,页面刷新后这个token就看不见了

回到git bash 继续提交,在githup登陆弹出框中输入账号密码,第一次输入的是你githup的账号密码,第二次弹出后输入git账号,密码换成刚刚生成的token.

如果两次错误,会提示你在git bash中输入账号,之后会弹出一个密码框,这个也是输入token

总之,账号还是输入你自己的git账号,密码,第二次之后输入token