---
layout: post
title: Github访问不了、不稳定？教你轻松解决
date: 2022-02-19
tags: github
---    

## 修改hosts文件
先找到 hosts 文件的位置，不同操作系统，hosts 文件的存储位置也不同：

Windows 系统：C:\Windows\System32\drivers\etc\hosts
Linux 系统：/etc/hosts
Mac（苹果电脑）系统：/etc/hosts
Android（安卓）系统：/system/etc/hosts
iPhone（iOS）系统：/etc/hosts

打开 hosts 文件，添加一行，将 xx 替换为你查询到的解析地址即可：
```bash
# GitHub520 Host Start
185.199.108.154               github.githubassets.com
140.82.113.22                 central.github.com
185.199.108.133               desktop.githubusercontent.com
185.199.108.153               assets-cdn.github.com
185.199.108.133               camo.githubusercontent.com
185.199.108.133               github.map.fastly.net
199.232.69.194                github.global.ssl.fastly.net
140.82.113.4                  gist.github.com
185.199.108.153               github.io
140.82.114.3                  github.com
140.82.112.6                  api.github.com
185.199.108.133               raw.githubusercontent.com
185.199.108.133               user-images.githubusercontent.com
185.199.108.133               favicons.githubusercontent.com
185.199.108.133               avatars5.githubusercontent.com
185.199.108.133               avatars4.githubusercontent.com
185.199.108.133               avatars3.githubusercontent.com
185.199.108.133               avatars2.githubusercontent.com
185.199.108.133               avatars1.githubusercontent.com
185.199.108.133               avatars0.githubusercontent.com
185.199.108.133               avatars.githubusercontent.com
140.82.114.10                 codeload.github.com
52.217.107.92                 github-cloud.s3.amazonaws.com
52.216.238.99                 github-com.s3.amazonaws.com
52.216.178.187                github-production-release-asset-2e65be.s3.amazonaws.com
52.217.101.68                 github-production-user-asset-6210df.s3.amazonaws.com
52.217.48.84                  github-production-repository-file-5c1aeb.s3.amazonaws.com
185.199.108.153               githubstatus.com
64.71.168.201                 github.community
185.199.108.133               media.githubusercontent.com


# Update time: 2021-03-18T10:33:43+08:00
# Star me GitHub url: https://github.com/521xueweihan/GitHub520
# GitHub520 Host End
```


## 最新host获取：
https://gitee.com/yunplusplus/GitHub520
