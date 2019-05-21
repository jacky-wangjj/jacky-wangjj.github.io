---
layout: post
title: 关系型数据库导出数据到非关系型数据库
date: 2019-05-17
tags: 大数据
---  
### sqoop定时增量导入mysql数据到hive

hive表结构中的数据类型与mysql对应如下
```
MySQL(bigint) --> Hive(bigint)
MySQL(tinyint) --> Hive(tinyint)
MySQL(int) --> Hive(int)
MySQL(double) --> Hive(double)
MySQL(bit) --> Hive(boolean)
MySQL(varchar) --> Hive(string)
MySQL(decimal) --> Hive(double)
MySQL(date/timestamp) --> Hive(string)
```
`--map-column-hive cost="DECIMAL",date="DATE"`    
在导入时通过`--map-column-hive`作出映射关系指定，可以修改默认类型。    

sqoop测试
```shell
sqoop list-databases --connect jdbc:mysql://10.110.181.39/ --username root -P
```

#### 创建sqoop job      
一种方式是只会将数据放入指定目录不会创建hive表，可以实现增量导入，需要自己创建外部表。      

- 创建外部表
```sql
CREATE EXTERNAL TABLE `t3`(`id` int, `addr` string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n' LOCATION 'hdfs://10.110.181.40:8020/apps/hive/warehouse/t3'
```

- 使用sqoop增量导入数据到hdfs指定目录    
```shell
sqoop job --create hivejob -- import \
  --connect jdbc:mysql://10.110.181.39:3306/test?useCursorFetch=true \
  --username root \
  --password-file /tmp/sqoop/39mysql.pwd \
  --target-dir /apps/hive/warehouse/t3 \
  --table t3 \
  --fields-terminated-by "," \
  --lines-terminated-by "\n" \
  --null-string '\\N' \
  --null-non-string '\\N' \
  --hive-import \
  --hive-table t3 \
  --incremental append \
  --check-column id \
  -m 1
```

另外一种方式，sqoop导入数据的时候自动创建hive表，若有表则增量导入。    
```shell
sqoop job --create hivejob -- import \
   --connect jdbc:mysql://10.110.181.39:3306/test?useCursorFetch=true \
   --username root \
   --password-file /tmp/sqoop/39mysql.pwd \
   --target-dir /apps/hive/warehouse/t3 \
   --table t3 \
   --fields-terminated-by "," \
   --lines-terminated-by "\n" \
   --null-string '\\N' \
   --null-non-string '\\N' \
   --hive-import \
   --hive-table t3 \
   --create-hive-table \
   --incremental append \
   --check-column id \
   -m 1
```

要使sqoop job中的`--password`起作用需添加如下配置：     
```xml
<property>
    <name>sqoop.metastore.client.record.password</name>
    <value>true</value>
    <description>If true, allow saved passwords in the metastore.
    </description>
</property>
```

可使用`--password-file /tmp/sqoop/39mysql.pwd`代替`--password`参数，尽量不使用`--password`，ps命令可以查看到此参数。     
[注意：39mysql.pwd需要修改400权限](https://www.jianshu.com/p/f77af0257b80)。     
```shell
echo -n "bigdata" > 39mysql.pwd
hdfs dfs -put 39mysql.pwd /tmp/sqoop/
hdfs dfs -chmod 400 /tmp/sqoop/39mysql.pwd
```

#### 删除sqoop job
```shell
sqoop job -delete hivejob
```

#### 执行sqoop job
```shell
sqoop job -exec hivejob
```

创建sqoop-hive.sh
```shell
su hive
/usr/bin/sqoop job -exec hivejob >> /tmp/sqoop_hive_`date`.log 2>&1
```

创建crontab定时任务: `crontab -u hive -e`
```shell
# 每天0点0分增量导入mysql数据到hive
0 0 * * * /bin/sh /home/wangjj17/sqoop-hive.sh
```

启动cron服务
```shell
service crond start    //启动服务
service crond status   //查看服务状态
```

执行`crontab sqoop_to_hive`提交给cron进程。     
**crontab文件的含义：**    
用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：    
minute hour day month week command     
其中：     
minute： 表示分钟，可以是从0到59之间的任何整数。     
hour：表示小时，可以是从0到23之间的任何整数。     
day：表示日期，可以是从1到31之间的任何整数。     
month：表示月份，可以是从1到12之间的任何整数。     
week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。     
command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。     

sqoop支持的增量模式：     
Sqoop supports two types of incremental imports: append and lastmodified. You can use the --incremental argument to specify the type of incremental import to perform.

You should specify append mode when importing a table where new rows are continually being added with increasing row id values. You specify the column containing the row’s id with --check-column. Sqoop imports rows where the check column has a value greater than the one specified with --last-value.

An alternate table update strategy supported by Sqoop is called lastmodified mode. You should use this when rows of the source table may be updated, and each such update will set the value of a last-modified column to the current timestamp. Rows where the check column holds a timestamp more recent than the timestamp specified with --last-value are imported.  
  --check-column col：控制增量的变量字段，这个字段最好不是字符串类型的。    
  --incremental mode：增量模式选择，append或lastmodified。     
  --last-value 0：从哪里开始导入。     
若指定--incremental lastmodified，则--check-column需指定一个timestamp字段，并且该字段应该在数据修改时，同时被更新。     

sqoop导入hive分区表     
https://stackoverflow.com/questions/42991952/hadoop-sqoop-export-import-partitioned-table     

问题：     
增量导出，若导出后数据被修改，如何保持同步？
cron定时任务，最后一条数据重复导入？    

https://www.jianshu.com/p/f202ee34d1e3      
https://www.jianshu.com/p/bb78ccd0252f      
https://blog.csdn.net/ggz631047367/article/details/50185591

#### sqoop --option-file形式
`sqoop --option-file`形式不支持增量导入，只有sqoop job形式可记录导入数据。     
创建sqoop配置文件hivejob，覆盖hive表中数据。     
```
import
--connect
jdbc:mysql://10.110.181.39:3306/test?useCursorFetch=true
--username
root
--password-file
/tmp/sqoop/39mysql.pwd
--table
t3
--fields-terminated-by
","
--lines-terminated-by
"\n"
--null-string
'\\N'
--null-non-string
'\\N'
--hive-import
--hive-overwrite
--create-hive-table
--hive-table
t3
-m
1
```

执行sqoop配置文件中的内容
```shell
/usr/bin/sqoop --options-file /home/wangjj17/sqoop/hivejob >> /tmp/sqoop_hive_`date`.log 2>&1
```

### sqoop定时增量导入mysql数据到hbase
切换hive用户
```shell
su hdfs
```

进入hbase shell给hdfs用户分配建表权限
```
grant 'hdfs','RWCA'
```

#### 创建sqoop job
```shell
sqoop job --create hbasejob -- import \
  --connect jdbc:mysql://10.110.181.39:3306/test?useCursorFetch=true \
  --username root \
  --password-file /tmp/sqoop/39mysql.pwd \
  --table t3 \
  --columns ID,ADDR \
  --hbase-table t3 \
  --column-family cf \
  --hbase-row-key ID \
  --hbase-create-table \
  --incremental append \
  --check-column ID \
  -m 1
```

#### 执行sqoop job
创建sqoop-hbase.sh
```shell
su hdfs
/usr/bin/sqoop job -exec hbasejob >> /tmp/sqoop_hbase_`date`.log 2>&1
```

创建定时任务：`crontab -u hdfs -e`
```shell
# 每天0点0分增量导入mysql数据到hbase
0 0 * * * /bin/sh /home/wangjj17/sqoop-hbase.sh
```

启动cron服务
```shell
service crond start    //启动服务
service crond status   //查看服务状态
```

若想定制hbase的rowkey，可在执行sqoop导入命令之前，创建临时表拷贝原始表，添加hbase-row-key列，并按照定制rowkey规则拼接rowkey插入到临时表中，最后执行sqoop命令导入临时表中的数据，并指定--hbase-row-key hbase-row-key。    

参考链接：     
https://www.cnblogs.com/sxt-zkys/p/7240041.html
https://blog.csdn.net/u012965373/article/details/53183498      
https://www.zybuluo.com/aitanjupt/note/209968           
https://www.cnblogs.com/klb561/p/9085615.html       
