---
layout: post
title: Mysql常用操作
date: 2019-06-09
tags: mysql
---  
### centos7安装mysql
- 配置yum源      
在mysql官网下载yum源rpm安装包：http://dev.mysql.com/downloads/repo/yum/

1) 下载mysql源安装包     
```shell
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

2) 安装mysql源     
```shell
yum localinstall mysql80-community-release-el7-3.noarch.rpm
```

可以修改`/etc/yum.repos.d/mysql-community.repo`源，改变默认安装的mysql版本。比如要安装5.6版本，则将5.7源的enabled=1改成enabled=0。然后再讲5.6元的enabled=0改为enabled=1即可。

- 安装mysql      
```shell
yum install mysql-community-server
```
或者先下载安装包再本地安装，官网地址：https://dev.mysql.com/downloads/mysql/
```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-community-server-8.0.16-2.el7.x86_64.rpm
yum localinstall mysql-community-server-8.0.16-2.el7.x86_64.rpm
```

- 启动mysql服务      
```shell
systemctl start mysqld
# 查看mysql的启动状态
systemctl status mysqld
```

- 设置开机启动
```shell
systemctl enbale mysqld
systemctl restart mysqld
```

- 修改root本地登录密码       
mysql安装完成后，在/var/log/mysqld.log文件中给root生成了一个默认密码。      
1) 通过登录mysql进行密码修改，这种方式需要先修改一个服务规则的密码如'123456abc!'，然后配置密码策略。
```mysql
mysql> show variables like '%password%';
set global validate_password.length=6;
set global validate_password.policy=LOW;
alter user 'root'@'localhost' identified by '123456';
```

2) 通过修改配置文件`/etc/my.cnf`，修改密码。
```
# 选择0（LOW），1（MEDIUM），2（STRONG）其中一种，选择2需要提供密码字典文件
validate_password_policy=0
# 如果不需要密码策略，添加my.cnf文件中添加如下配置禁用即可：
validate_password = off
```
重新启动mysql服务使配置生效：`systemctl restart mysqld`

- 添加远程登录用户      
```mysql
mysql> grant all privileges on *.* to 'user'@'%' identified by '123456' with grant option;
```

- 配置默认编码为utf8，修改配置文件`/etc/my.cnf`      
```
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
```
重启mysqld生效。     

- 默认配置文件路径       
配置文件：/etc/my.cnf
日志文件：/var/log//var/log/mysqld.log
服务启动脚本：/usr/lib/systemd/system/mysqld.service
socket文件：/var/run/mysqld/mysqld.pid

### mysql常用用户操作
- 登录     
```sql
mysql -u root -p
```

- 创建用户         
Host字段是‘localhost’表示本地登录，不能在另外一台机器上远程登录；若想远程登录可以使用‘%’。也可以指定某个IP可以登录。      
```sql
insert into mysql.user(Host, User, Password) values("%", "user", password("123456"));
```

- 删除用户
```sql
delete from user where User='user' and Host='%';
flush privileges;
```

### mysql常用权限操作
- 授权user用户拥有testDB数据库的所有权限           
格式：grant 权限 on 数据库.* to 用户名@登录主机 identified by '密码'     
```sql
grant all privileges on testDB.* to 'user'@'%' identified by '123456';
flush privileges;
```

### mysql常用查询操作
- 列出所有数据库       
`show databases;`    

- 切换数据库       
`use testDB;`     

- 显示数据表结构      
`describe tablename;`       

- 删除数据库和数据表      
`drop databese databasename;`      
`drop table tablename;`       


#### 单表操作
- 条件查询         
使用where关键字。
```sql
select * from role where id=1 and name='1';
```

- 模糊查询         
使用like关键字。加SQL的通配符。         

| 符号   | 说明                     |
| :---: | ------------------------ |
| %     | 匹配任意0个或多个字符      |
| _     | 匹配任意单个字符           |
| [A-Z] | 从A到Z的任何单个字母       |
| [^C]  | 不包含C，不包含括号所列字符 |

```sql
select * from role where name like '1%';
```
查询内容包含通配符（% _ []）时，需要把特殊字符使用`[]`括起来。      

- 多字段排序        
order by多个字段排序使用逗号分隔每一个字段，每一个字段后面跟排序方式（ASC/DESC），若不指明排序方式，默认是升序（ASC）；排序先按第一个字段排序，若相同再按后续字段一次排序。       

```sql
SELECT * FROM `role` order by description desc, type asc;
```
![](https://jacky-wangjj.github.io/images/blog/mysql/mysql-multi-field-sort-1.png#pic_center)

```sql
SELECT * FROM `role` order by description asc, type desc;
```
![](https://jacky-wangjj.github.io/images/blog/mysql/mysql-multi-field-sort-2.png#pic_center)

#### 多表操作
- 表的别名         
1）别名通常是一个缩短了的表名，用于在连接中引用表中的特定列，如果连接中的多个表中有相同的列名称存在，必须用表名或表的别名限定列名。       
2）如果定义了表的别名就不能再使用表名了。       

第一种方式通过关键字AS指定：     
```sql
SELECT a.id,a.name,a.address,b.math,b.english,b.chinese FROM tb_demo065 AS a,tb_demo065_tel AS b WHERE a.id=b.id;
```

第二种方式是在表名后直接加表的别名实现：
```sql
SELECT a.id,a.name,a.address,b.math,b.english,b.chinese FROM tb_demo065  a,tb_demo065_tel  b WHERE a.id=b.id;
```

- 简单嵌套查询      
子查询是一个select查询，返回单个值且嵌套在select、insert、update和delete语句或其他查询语句中。       
内连接：把查询结果作为where子句的查询条件。       
```sql
select id, name, sex, date from tb where id in (select id from tb_class where class=1);
```

- 内连接INNER JOIN       
```sql
select filedlist from table1 [inner] join table2 on table1.column1=table2.column1;
```
filedlist是要显示的字段，inner表示表之间的连接方式是内连接，table1.column1=table2.column1用于指明两表间的连接条件。     

- 左外连接LEFT OUTER JOIN         
左外连接可以简写成LEFT JOIN，它以左侧表为基准。左侧表中所有信息将被全部输出，而右侧表信息则只会输出符合条件的信息，对于不符合条件的信息则返回NULL。     

- 右外连接RIGHT OUTER JOIN
右外连接可以简写成RIGHT JOIN，它以右侧的表为基准。右侧表中所有信息将被全部输出，而左侧表信息则只输出符合条件的信息，对于不符合条件的信息则返回NULL。     

- IN或NOT IN限定范围      
利用IN可指定在范围内查询，NOT IN可指定在范围外查询。     

- HAVING语句过滤分组数据      
HAVING用于指定组或聚合的搜索条件，HAVING通常与GROUP BY语句一起使用，如果不含GROUP BY子句，则HAVING的行为和WHERE子句一样。      
```sql
SELECT name,math FROM tb_demo083 GROUP BY id HAVING math > '95';
```
