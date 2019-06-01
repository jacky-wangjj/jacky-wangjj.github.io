---
layout: post
title: idea永久破解及基本配置
date: 2019-06-01
tags: 环境
---  
### idea下载与安装
官网下载连接：https://www.jetbrains.com/idea/download/previous.html       
2018.3.6 for Windows ZIP Archive (zip)       
解压zip包到指定位置即可      

### idea破解
#### 一般破解
添加hosts配置       
C -> Windows -> System32 -> drivers -> etc -> hosts 添加如下配置
```
0.0.0.0 account.jetbrains.com
0.0.0.0 www.jetbrains.com
```

浏览器打开`http://idea.lanyus.com/`        
![](https://jacky-wangjj.github.io/images/blog/env/idea-lanyus-com.png#pic_center)

获取注册码，填入idea的activation-code框中

#### 永久破解
下载JetbrainsIdesCrack-4.2.jar        
链接: https://pan.baidu.com/s/1VH2uxFzxWpllmdjzF82GRg 提取码: r9mj       

修改bin/idea.exe.vmoptions和bin/idea64.exe.vmoptions，添加如下内容：         
```
-javaagent:D:\JavaEE\DevelopTools\ideaIU-2018\JetbrainsIdesCrack-4.2.jar
```

activation-code框中输入`Deshun`，即可激活。     

### idea基本配置
- 配置.IntelliJIdea目录到非C盘位置。        
修改bin/idea.properties文件中相关配置           
```xml
  #idea.config.path=${user.home}/.IntelliJIdea/config
  idea.config.path=D:/JavaEE/DevelopTools/.IntelliJIdea2018/config

  #idea.system.path=${user.home}/.IntelliJIdea/system
  idea.system.path=D:/JavaEE/DevelopTools/.IntelliJIdea2018/system

  #idea.plugins.path=${idea.config.path}/plugins
  idea.plugins.path=D:/JavaEE/DevelopTools/.IntelliJIdea2018/plugins

  # idea.log.path=${idea.system.path}/log
  idea.log.path=D:/JavaEE/DevelopTools/.IntelliJIdea2018/log
```
重启idea

- 配置idea默认主题        
打开Configure -> Settings       
具体设置参考https://blog.csdn.net/dear_Alice_moon/article/details/78123527       

- 配置idea默认maven及maven仓库           
打开项目默认设置：Configure -> Projects Defaults -> Settings            
![](https://jacky-wangjj.github.io/images/blog/env/idea-default-setting.png#pic_center)

Build,Excution,Deployment -> Build Tools -> Maven         
![](https://jacky-wangjj.github.io/images/blog/env/idea-maven-setting.png#pic_center)

修改maven/conf/settings.xml        
```xml
<localRepository>D:/JavaEE/DevelopTools/repository</localRepository>

<mirror>
   <id>aliyun</id>
   <name>aliyun Maven</name>
   <mirrorOf>*</mirrorOf>
   <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>
```

- 配置git          
Version Control -> Git         
![](https://jacky-wangjj.github.io/images/blog/env/idea-git-setting.png#pic_center)

- 添加常用插件          
![](https://jacky-wangjj.github.io/images/blog/env/idea-plugins.png#pic_center)

- 配置代码模板        
Editor -> File and Code Templates        

Class
```java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
/**
 * @Author wangjj17@lenovo.com
 * @Date ${DATE}
 */
public class ${NAME} {

    public static void main(String[] args) {

    }
}
```

Interface
```java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
/**
 * @Author wangjj17@lenovo.com
 * @Date ${DATE}
 */
 public interface ${NAME} {

 }
```

Enum
```java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
/**
 * @Author wangjj17@lenovo.com
 * @Date ${DATE}
 */
 public enum ${NAME} {

 }
```

- 添加代码模板mybatis-config         
Editor -> File and Code Templates
![](https://jacky-wangjj.github.io/images/blog/env/idea-mybatis-config.png#pic_center)

- 配置编码utf-8            
Editor -> FileEncodings      
![](https://jacky-wangjj.github.io/images/blog/env/idea-file-encoding.png#pic_center)
