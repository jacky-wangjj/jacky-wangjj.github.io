---
layout: post
title: MongoDB安装与使用实例
date: 2019-05-23
tags: 大数据
---  
### MongoDB安装
官网下载：https://www.mongodb.com/download-center           
下载连接：https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.9.tgz          

##### 解压mongodb-linux-x86_64-4.0.9.tgz，配置环境变量         
```shell
tar -zxvf mongodb-linux-x86_64-4.0.9.tgz
vim ~/.bashrc
# 添加如下内容
export MONGODB_HOME=/home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9
export PATH=$MONGODB_HOME/bin:$PATH

source ~/.bashrc
```

##### 查看mongodb版本信息`mongod -v`
```shell
[root@node16 mongodb]# mongod -v
2019-05-23T10:40:50.743+0800 D NETWORK  [main] fd limit hard:65535 soft:65535 max conn: 52428
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten] MongoDB starting : pid=88634 port=27017 dbpath=/data/db 64-bit host=node16.sleap.com
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten] db version v4.0.9
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten] git version: fc525e2d9b0e4bceff5c2201457e564362909765
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten] allocator: tcmalloc
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten] modules: none
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten] build environment:
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten]     distarch: x86_64
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten]     target_arch: x86_64
2019-05-23T10:40:50.743+0800 I CONTROL  [initandlisten] options: { systemLog: { verbosity: 1 } }
2019-05-23T10:40:50.743+0800 D NETWORK  [initandlisten] fd limit hard:65535 soft:65535 max conn: 52428
2019-05-23T10:40:50.743+0800 D -        [initandlisten] User Assertion: NonExistentPath: Data directory /data/db not found. src/mongo/db/storage/storage_engine_init.cpp 243
2019-05-23T10:40:50.743+0800 I STORAGE  [initandlisten] exception in initAndListen: NonExistentPath: Data directory /data/db not found., terminating
2019-05-23T10:40:50.743+0800 D -        [initandlisten] User Assertion: NotMaster: not primary so can't step down src/mongo/db/db.cpp 893
2019-05-23T10:40:50.743+0800 I NETWORK  [initandlisten] shutdown: going to close listening sockets...
2019-05-23T10:40:50.743+0800 I NETWORK  [initandlisten] removing socket file: /tmp/mongodb-27017.sock
2019-05-23T10:40:50.744+0800 I CONTROL  [initandlisten] now exiting
2019-05-23T10:40:50.744+0800 I CONTROL  [initandlisten] shutting down with code:100
```

##### 创建数据库目录       
```shell
mkdir -p /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/data/mongodb
mkdir -p /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/data/mongodb/log
touch /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/data/mongodb/log/mongodb.log
```

##### 添加配置文件       
新建mongodb.conf配置文件，通过这个配置文件进行启动。        
```shell
vim /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/mongodb.conf
```

配置文件内容如下
```shell
dbpath=/home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/data/mongodb
logpath=/home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/data/mongodb/log/mongodb.log
logappend=true
bind_ip=0.0.0.0 #允许所有IP远程访问
port=27017
fork=true
##auth = true # 先关闭, 创建好用户在启动
```

##### 通过配置文件启动
```shell
mongod -f /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/mongodb.conf
# 出现如下内容表示启动成功
about to fork child process, waiting until server is ready for connections.
forked process: 93988
child process started successfully, parent exiting
```

##### 配置防火墙
```shell
vim /etc/sysconfig/iptables
# 添加如下内容
-A INPUT -m state --state NEW -m tcp -p tcp --dport 27017 -j ACCEPT
# 重启iptables
service iptables restart
```

##### 进入MongoDB控制台
```shell
mongo
> use admin #进入admin数据库
> db.shutdownServer() #关闭MongoDB数据库
> exit #退出
```

添加普通用户
```shell
use test #创建数据库
# 创建普通用户，设置权限
db.createUser({
    user: "test",
    pwd: "test",
    roles: [{role: "readWrite", db: "test"}]
})
```

添加管理员账号（root权限）
```shell
db.createUser({
    user:"root",
    pwd:"123456",
    roles:[{role:"root",db:"admin"}]
})
```

查看所有用户
```shell
db.system.users.find()
```

关闭MongoDB
```shell
mongo 127.0.0.1:27017/admin --eval "db.shutdownServer()"  #关闭MongoDB
```

##### 设置开机自启动
```shell
vim /etc/rc.d/init.d/mongod

chmod +x /etc/rc.d/init.d/mongod  #添加可执行权限
chkconfig mongod on #设置开机启动
service mongod start #启动MongoDB
```

`/etc/rc.d/init.d/mongod`文件内容如下
```shell
#!/bin/sh
# chkconfig: - 64 36
# description:mongod

start() {  
  /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/bin/mongod  --config /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/mongodb.conf #/usr前面不要有空格
}  

stop() {  
  /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/bin/mongod --config /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/mongodb.conf --shutdown  
}
status() {  
  /home/wangjj17/mongodb/mongodb-linux-x86_64-4.0.9/bin/mongo 127.0.0.1:27017/admin --eval "db.stats()"
}
case "$1" in
  start)
    echo -e "Starting mongod"
    start
    ;;
  stop)
    echo -e "Stopping mongod"
    stop
    ;;
  restart)
    echo -e "Restarting mongod"
    stop
    start
    ;;
  status)
    echo -e "Mongod status"
    status
    ;;
  *)
    echo -e "Usage: $0 {start|stop|status|restart}"
    exit 1
esac
exit $?
```

### MongoDB常用命令
官方文档：https://docs.mongodb.com/manual/
##### 查看当前用户
```shell
show users;
```

##### 用户创建
先关闭auth认证
```shell
> use admin
switched to db admin
> db.createUser({
... user: "dba",
... pwd: "dba",
... roles: [{role: "userAdminAnyDatabase", db: "admin"}]
... })
Successfully added user: {
	"user" : "dba",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}

```
user: 用户名      
pwd: 密码         
roles: 指定用户的角色           
[role里的角色可以选如下内容](https://docs.mongodb.com/manual/reference/built-in-roles/#built-in-roles)           
```shell
1. 数据库用户角色：read、readWrite;
2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4. 备份恢复角色：backup、restore；
5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6. 超级用户角色：root  
// 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
7. 内部角色：__system

Read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，超级权限
```

创建普通用户
```shell
> use testdb
switched to db testdb
> db.createUser({
... user: "test",
... pwd: "test",
... roles: [{role: "readWrite", db: "testdb"}]
... })
Successfully added user: {
	"user" : "test",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "testdb"
		}
	]
}
```

删除用户
```shell
> use testdb
switched to db testdb
> db.dropUser("dba")
true
```

用户验证登录
```shell
mongo --port 27017 -u "test" -p "test" --authenticationDatabase "testdb"
MongoDB shell version v4.0.9
connecting to: mongodb://127.0.0.1:27017/?authSource=testdb&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("4b07f432-e98a-4c8d-9752-87bf481f9289") }
MongoDB server version: 4.0.9
```

另一种登录验证方式
```shell
[root@node16 wangjj17]# mongo
MongoDB shell version v4.0.9
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("634419c1-fe46-4863-97be-8255681ed66b") }
MongoDB server version: 4.0.9
> use admin
switched to db admin
> db.auth("dba", "dba")
1 #输出1表示验证成功
```

##### 创建数据库
```shell
> use testdb
switched to db testdb
> db #查看当前所使用的数据库
testdb
> show dbs #查看所有数据库
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
> show collections #查看当前数据库所有集合（表格）
col
```

##### 删除数据库
```shell
use testdb
> db.dropDatabase()
{ "dropped" : "testdb", "ok" : 1 }
```

##### 向testdb数据库插入数据
```shell
> db.testdb.insert({x: 10})
WriteResult({ "nInserted" : 1 })
```

##### 查看数据
```shell
> db.testdb.find()
{ "_id" : ObjectId("5ce6502c61d1499b42f3e9af"), "x" : 10 }
```

##### 插入文档
MongoDB使用insert()或save()方法向集合中插入文档
```shell
> use testdb
switched to db testdb
> document=({title: 'test title',
... description: 'test description',
... url: 'www.lenovo.com',
... tags: ['mongodb', 'database', 'NoSQL'],
... likes: 100
... })
{
	"title" : "test title",
	"description" : "test description",
	"url" : "www.lenovo.com",
	"tags" : [
		"mongodb",
		"database",
		"NoSQL"
	],
	"likes" : 100
}
> db.col.insert(document)
WriteResult({ "nInserted" : 1 })
> db.col.find()
{ "_id" : ObjectId("5ce65ffb61d1499b42f3e9b0"), "title" : "test title", "description" : "test description", "url" : "www.lenovo.com", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 }
```
其中`col`是集合名，如果该集合不在数据库中，MongoDB会自动创建该集合并插入文档。      
插入文档也可以使用`db.col.save(document)`命令，如果不指定`_id`字段，则自动生成。      

`db.collection.insertOne()`：向指定集合中插入一条文档数据。      
`db.collection.inserMany()`：向指定集合中插入多条文档数据。     

##### 删除文档
删除集合下全部文档
```shell
> db.col.deleteMany({})
{ "acknowledged" : true, "deletedCount" : 2 }
> db.col.find()
>
```

删除指定文档
```shell
> db.col.insert({weixin:"souyunku"})
WriteResult({ "nInserted" : 1 })
> db.col.find()
{ "_id" : ObjectId("5a69f5a0ec3046ee8ae54dc1"), "weixin" : "souyunku" }
> db.col.deleteOne({weixin:"souyunku"})
{ "acknowledged" : true, "deletedCount" : 1 }
> db.col.find()
>
```

##### 查询文档
`find()`方法以非结构化的方式来显示所有文档
```shell
> db.collection.find(query, projection)
```
query ：可选，使用查询操作符指定查询条件。     
projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。     

`pretty()`以格式化的方式来显示所有文档
```shell
> db.col.find().pretty()
{
	"_id" : ObjectId("5ce65ffb61d1499b42f3e9b0"),
	"title" : "test title",
	"description" : "test description",
	"url" : "www.lenovo.com",
	"tags" : [
		"mongodb",
		"database",
		"NoSQL"
	],
	"likes" : 100
}
```

MongoDB与RDBMS Where语句比较
![](![](https://jacky-wangjj.github.io/images/blog/bigdata/mongodb-vs-rdbms-where.png#pic_center)      

MongoDB的`find()`方法可以出入多个键，每个键以逗号隔开，类似常规SQL的AND条件查询。      
```shell
> db.col.find({key1:value1, key2:value2}).pretty()
```

MongoDB OR 条件语句使用了关键字 $or,语法格式如下：     
```shell
>db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

##### MongoDB条件操作符     
大于操作符 - $gt
```shell
db.col.find({"likes" : {$gt : 100}})

select * from col where likes > 100;
```

大于等于操作符 - $gte
```shell
db.col.find({likes : {$gte : 100}})

select * from col where likes >=100;
```

小于操作符 - $lt
```shell
db.col.find({likes : {$lt : 150}})

select * from col where likes < 150;
```

小于操作符 - $lte
```shell
db.col.find({likes : {$lte : 150}})

select * from col where likes <= 150;
```

使用 (<) 和 (>) 查询 - lt 和 gt
```shell
db.col.find({likes : {$lt :200, $gt : 100}})

select * from col where likes>100 AND  likes<200;
```

##### 排序
在 MongoDB 中使用使用 sort() 方法对数据进行排序，sort() 方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。     
```shell
db.collection.find().sort({KEY:1})
```

##### 更新数据
update() 方法用于更新已存在的文档。语法格式如下：
```shell
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```
query：对哪些数据进行更新操作。      
update：对这些数据做什么操作。         
upsert（可选）：如果不存在update的记录，是否将其作为记录插入。true为插入，默认是false，不插入。        
multi（可选）：是否更新多条记录。MongoDb 默认是false，只更新找到的第一条记录。如果这个参数为true,就把按条件查出来多条记录全部更新。        
writeConcern（可选）：表示抛出异常的级别。      

### 使用Java编程连接MongoDB
生产中URL形式对数据库进行连接
```shell
mongodb://10.110.181.40:27017/testdb
```

添加用户名密码验证
```shell
mongodb://user:password@10.110.181.40:27017/testdb
```

##### java操作MongoDB实例
添加pom依赖
```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.10.0</version>
</dependency>
```

java源码如下
```java
import com.mongodb.*;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.Filters;
import org.bson.Document;

import java.util.ArrayList;
import java.util.List;

/**
 * @ClassName MongoDBUtils
 * @Description TODO
 * @Author wangjj
 * @Date 2019/5/23 19:35
 */
public class MongoDBUtils {

    public static void main(String[] args) {
        // 使用credential
        MongoClientOptions options = MongoClientOptions.builder()
                //设置主机最大连接数
                .connectionsPerHost(50)
                //设置连接超时时间为10s
                .connectTimeout(1000*10)
                //设置最长等待时间为10s
                .maxWaitTime(1000*10)
                .build();
        MongoCredential credential = MongoCredential.createCredential("test", "testdb", "test".toCharArray());
        MongoClient mongoClient = new MongoClient(new ServerAddress("10.110.181.40", 27017), credential, options);
/*
        // 使用连接URL
        MongoClientURI uri = new MongoClientURI("mongodb://test:test@10.110.181.40/?authSource=testdb");
        MongoClient mongoClient = new MongoClient(uri);
*/
        MongoDatabase database = mongoClient.getDatabase("testdb");
        System.out.println("MongoDatabase inof is : "+database.getName());
        // 列出所有集合
        System.out.println("List Collections: ");
        for (String collection : database.listCollectionNames()) {
            System.out.println(collection);
        }
        // 插入文档
        MongoCollection<Document> col = database.getCollection("col");
        Document document = new Document("_id", 1).append("title", "MongoDB insert demo")
                .append("description", "database")
                .append("likes", 30)
                .append("by", "wangjj")
                .append("url", "https:jacky-wangjj.github.io");
        col.insertOne(document);
        // 插入多行
        Document doc1 = new Document("_id", 2).append("title", "MongoDB insert demo")
                .append("description", "database")
                .append("likes", 30)
                .append("by", "wangjj")
                .append("url", "https:jacky-wangjj.github.io");
        Document doc2 = new Document("_id", 3).append("title", "MongoDB insert demo")
                .append("description", "database")
                .append("likes", 30)
                .append("by", "wangjj")
                .append("url", "https:jacky-wangjj.github.io");
        List<Document> docs = new ArrayList<Document>();
        docs.add(doc1);
        docs.add(doc2);
        col.insertMany(docs);
        // 更新查到的一条文档
        col.updateOne(Filters.eq("_id", 1), new Document("$set", new Document("by", "jacky")));
        // 更新查到的所有文档
        col.updateMany(Filters.eq("_id", 1), new Document("$set", new Document("by", "jacky")));
        // 删除查到的一条文档
        col.deleteOne(Filters.eq("by", "jacky"));
        // 删除查到的所有文档
        col.deleteMany(Filters.eq("by", "wangjj"));
        // 查寻所有文档
        FindIterable<Document> documents = col.find();
        for (Document doc : documents) {
            System.out.println(doc.toJson());
        }
        // 按条件查询
        FindIterable<Document> documents1 = col.find(Filters.eq("by", "wangjj"));
        MongoCursor<Document> iterator = documents1.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next().toJson());
        }
    }
}
```
