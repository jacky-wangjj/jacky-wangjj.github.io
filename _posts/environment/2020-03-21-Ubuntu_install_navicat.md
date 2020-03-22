---
layout: post
title: ubuntu安装配置navicat
date: 2020-03-21
tags: 环境
---  
### 下载压缩包文件          
navicat120_premium_cs_x64.tar.gz              
navicat-keygen-for-x64.tar.xz      

### 安装navicat      
- 拷贝navicat120_premium_cs_x64.tar.gz至Ubuntu系统，并解压            
建议使用root用户

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322141731782.png)

- 修改start_navicat            

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322141955417.png)

- 启动./start_navicat                
启动后会到注册界面，不要关，下面开始破解

### 破解navicat          
- 安装wine        
sudo apt-get install wine          

- 配置/etc/hosts          
添加127.0.0.1       activate.navicat.com         

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142016813.png)

- 拷贝navicat-keygen-for-x64.tar.xz至Ubuntu系统并解压         

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142028495.png)

- 启动navicat-patcher.exe        
wine navicat-patcher.exe "/opt/navicat120_premium_cs_x64/Navicat" ./RegPrivateKey.pem

- 生成序列号active           
wine navicat-keygen.exe -text ./RegPrivateKey.pem

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142043265.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142053840.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-2020032214210031.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142106565.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142112415.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142119425.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142127981.png)

- 激活成功后显示如下：

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142136862.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322142207952.png)

大功告成！！！

### 中文乱码配置
- 修改start-navicat       
export LANG="zh_CN.UTF-8"       

- 修改start-navicat仍乱码      
安装 文泉驿字体     
```shell
sudo apt-get install ttf-wqy-microhei  #文泉驿-微米黑
sudo apt-get install ttf-wqy-zenhei  #文泉驿-正黑
sudo apt-get install xfonts-wqy #文泉驿-点阵宋体
```

设置navicat TOOLs Option 更改字体为  WenQuanYi Zen Hei Sharp即可解决，其中有三处字体需要更改      

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322144139436.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322144149476.png)

![](https://jacky-wangjj.github.io/images/blog/env/ubuntu-install-navicat/markdown-img-paste-20200322144203294.png)
