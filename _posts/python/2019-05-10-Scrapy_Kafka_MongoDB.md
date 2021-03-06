---
layout: post
title: Scrapy爬虫整合Kafka和MongoDB
date: 2019-05-10
tags: python爬虫
---  
### Scrapy安装
```shell
$ pip install Scrapy
```

### Scrapy项目
创建新的scrapy项目
```shell
$ scrapy startproject {ProjectName}
```

生成示例spider
```shell
$ cd {ProjectName}
$ scrapy genspider example example.com
```

测试
```shell
$ cd {ProjectName}
$ scrapy crawl {SpiderName}
保存爬取的信息到json文件
$ scrapy crawl {SpiderName} -o items.json -t json
```

PyCharm中执行和调试scrapy爬虫
新建begin.py，添加如下内容，然后执行begin.py     
```python
from scrapy import cmdline

if __name__ == "__main__":
    cmdline.execute("scrapy crawl stack".split())
```

### scrapy整合kafka
kafka常用命令    
- 创建topic
  ```shell
  kafka-topics --create --topic newtest --partitions 1 --replication-factor 1 --zookeeper localhost:2181
  ```

- 创建producer
  ```shell
  kafka-console-producer --broker-list localhost:9092 --topic newtest
  ```

- 创建consumer
  ```shell
  kafka-console-consumer --zookeeper localhost:2181 --topic newtest
  ```

kafka-python包安装
```shell
$ pip install kafka-python
```
[kafka-python官方资料](https://pypi.org/project/kafka-python/)     

pykafka包安装
```shell
pip install pykafka
```
[pykafka官方资料](https://pypi.org/project/pykafka/)    

pykafka依赖于librdkafka，因此需要安装librdkafka。[librdkafka源码项目](https://github.com/edenhill/librdkafka)     
```shell
yum install librdkafka-devel
安装包安装，安装包下载地址：https://centos.pkgs.org/6/epel-x86_64/lz4-r131-1.el6.x86_64.rpm.html
rpm -Uvh lz4-r131-1.el6.x86_64.rpm
rpm -Uvh librdkafka-0.11.6-1.el6.remi.x86_64.rpm
rpm -Uvh librdkafka-devel-0.11.6-1.el6.remi.x86_64.rpm
```

scrapy-kafka包安装
```shell
$ pip install scrapy-kafka
```
[scrapy kafka连接实例](https://github.com/dfdeshom/scrapy-kafka/tree/master/example)    

[Scrapy-Kafka-Demo](https://github.com/sunhailin-Leo/Scrapy-Kafka-Demo) 经测试可以使用        

[scrapy-kafka-redis](https://github.com/tenlee2012/scrapy-kafka-redis)    

### scrapy整合MongoDB
这里我们将所有元素保存到MongoDB的testdb数据库questions集合中。       
首先安装pymongo相关包
```shell
pip install bson
pip install pymongo
```

在settings指定相应的pipeline并添加一些数据库参数      
```python
ITEM_PIPELINES = {
   'scrapy_learning.pipelines.MongoDBPipeline': 300,
}
MONGODB_SERVER = '10.110.181.40'
MONGODB_PORT = 27017
MONGODB_DB = 'testdb'
MONGODB_COLLECTION = 'questions'
MONGODB_USER = 'test'
MONGODB_PWD = 'test'
```

在pipelines.py中添加数据存储相关代码
```python
import pymongo
from scrapy import log
from scrapy.exceptions import DropItem

class MongoDBPipeline(object):
    def __init__(self):
        connection = pymongo.MongoClient(settings.MONGODB_SERVER, settings.MONGODB_PORT)
        db = connection[settings.MONGODB_DB]
        db.authenticate(settings.MONGODB_USER, settings.MONGODB_PWD)
        self.collection = db[settings.MONGODB_COLLECTION]

    def process_item(self,item,spider):
        valid = True
        for data in item:
            if not data:
                valid = False
                raise DropItem("Missing{0}".format(data))
        if valid:
            self.collection.insert(dict(item))
            log.msg('question added to mongodb database!', level=log.DEBUG, spider=spider)
        return item
```

### 实例
新建项目scrapy_learning     
```shell
$ scrapy startproject scrapy_learning
```

项目结构如下：    
![](https://jacky-wangjj.github.io/images/blog/python/scrapy-project-structure.png)     

源码如下：    
scrapy_learning/items.py    
```python
import scrapy

class ScrapyLearningItem(scrapy.Item):
    # define the fields for your item here like:
    title = scrapy.Field()
    url = scrapy.Field()
    pass
```

scrapy_learning/settings.py
```python
BOT_NAME = 'scrapy_learning'

SPIDER_MODULES = ['scrapy_learning.spiders']
NEWSPIDER_MODULE = 'scrapy_learning.spiders'

ROBOTSTXT_OBEY = True

ITEM_PIPELINES = {
   'scrapy_learning.pipelines.ScrapyLearningPipeline': 300,
   'scrapy_learning.pipelines.MongoDBPipeline': 300,
}

# MongoDB相关参数配置
MONGODB_SERVER = '10.110.181.40'
MONGODB_PORT = 27017
MONGODB_DB = 'testdb'
MONGODB_COLLECTION = 'questions'
MONGODB_USER = 'test'
MONGODB_PWD = 'test'

# kafka配置
KAFKA_IP_PORT = ["10.110.181.40:6667"]
KAFKA_TOPIC = "scrapy_kafka"
```

scrapy_learning/pipelines.py
```python
import pymongo
from pykafka import KafkaClient
from scrapy import log
from scrapy.exceptions import DropItem
from scrapy.utils.serialize import ScrapyJSONEncoder

from scrapy_learning import settings


class ScrapyLearningPipeline(object):
    def __init__(self):
        kafka_ip_port = settings.KAFKA_IP_PORT
        kafka_topic = settings.KAFKA_TOPIC
        if len(kafka_ip_port) == 1:
            kafka_ip_port = kafka_ip_port[0]
        else:
            if isinstance(kafka_ip_port, list):
                kafka_ip_port = ",".join(kafka_ip_port)
            else:
                kafka_ip_port = kafka_ip_port
        self._client = KafkaClient(hosts=kafka_ip_port)
        self._producer = self._client.topics[kafka_topic.encode(encoding="UTF-8")].get_producer()
        self._encoder = ScrapyJSONEncoder()

    def process_item(self, item, spider):
        item = dict(item)
        item['spider'] = spider.name
        msg = self._encoder.encode(item)
        print(msg)
        self._producer.produce(msg.encode(encoding="UTF-8"))
        # self._producer.produce(item['url'].encode(encoding="UTF-8"))
        return item

    def close_spider(self,spider):
        self._producer.stop()

class MongoDBPipeline(object):
    def __init__(self):
        connection = pymongo.MongoClient(settings.MONGODB_SERVER, settings.MONGODB_PORT)
        db = connection[settings.MONGODB_DB]
        db.authenticate(settings.MONGODB_USER, settings.MONGODB_PWD)
        self.collection = db[settings.MONGODB_COLLECTION]

    def process_item(self,item,spider):
        valid = True
        for data in item:
            if not data:
                valid = False
                raise DropItem("Missing{0}".format(data))
        if valid:
            self.collection.insert(dict(item))
            log.msg('question added to mongodb database!', level=log.DEBUG, spider=spider)
        return item
```

scrapy_learning/spiders/example.py
```python
import scrapy
from scrapy import Selector

from scrapy_learning.items import ScrapyLearningItem

class ExampleSpider(scrapy.Spider):
    name = "stack"
    allowed_domains = ["stackoverflow.com"]
    start_urls = ['http://stackoverflow.com/questions?pagesize=50&sort=newest']

    def parse(self, response):
        questions = Selector(response).xpath('//div[@class="summary"]/h3')
        for question in questions:
            item = ScrapyLearningItem()
            item['title'] = question.xpath('a[@class="question-hyperlink"]/text()').extract()[0]
            item['url'] = question.xpath('a[@class="question-hyperlink"]/@href').extract()[0]
            yield item
```

scrapy.cfg
```
[settings]
default = scrapy_learning.settings

[deploy]
url = http://10.110.181.40:6800/
project = scrapy_learning
```

begin.py
```python
from scrapy import cmdline

if __name__ == "__main__":
    cmdline.execute("scrapy crawl stack".split())
```

### Scrapyd安装与使用
使用pip直接安装scrapyd和scrapyd-client
```shell
pip install scrapyd
pip install scrapyd-client
```
scrapyd默认配置修改       
/root/anaconda2/envs/wangjj17_3.6/lib/python3.6/site-packages/scrapyd/default_scrapyd.conf
```shell
[scrapyd]
eggs_dir    = /home/data/liebao/eggs
logs_dir    = /home/data/liebao/logs
items_dir   = /home/data/liebao/item
jobs_to_keep = 5
dbs_dir     = /home/data/liebao/dbs
max_proc    = 0
max_proc_per_cpu = 4
finished_to_keep = 100
poll_interval = 5.0
bind_address = 0.0.0.0
http_port   = 6800
debug       = off
runner      = scrapyd.runner
application = scrapyd.app.application
launcher    = scrapyd.launcher.Launcher
webroot     = scrapyd.website.Root

[services]
schedule.json     = scrapyd.webservice.Schedule
cancel.json       = scrapyd.webservice.Cancel
addversion.json   = scrapyd.webservice.AddVersion
listprojects.json = scrapyd.webservice.ListProjects
listversions.json = scrapyd.webservice.ListVersions
listspiders.json  = scrapyd.webservice.ListSpiders
delproject.json   = scrapyd.webservice.DeleteProject
delversion.json   = scrapyd.webservice.DeleteVersion
listjobs.json     = scrapyd.webservice.ListJobs
daemonstatus.json = scrapyd.webservice.DaemonStatus
```

版本问题：Scrapy==1.6.0 Twisted==18.9.0          
[scrapyd 问题 builtins.AttributeError: 'int' object has no attribute 'splitlines'](https://blog.csdn.net/qq_29719097/article/details/89431234)
```shell
pip list
pip uninstall Twisted
pip install Twisted==18.9.0
```

启动scrapyd服务
```shell
nohup scrapyd >> /var/scrapyd/scrapyd.log 2>&1 &
```

修改项目根目录scrapy.cfg
```
[deploy]
url = http://localhost:6800/
project = scrapy_learning
```

项目根目录执行scrapyd-deploy发布爬虫到scrapyd服务
```shell
scrapyd-deploy -p scrapy_learning
```

启动爬虫     
```shell
curl http://localhost:6800/schedule.json -d project=scrapy_learning -d spider=stack
```

可以在浏览器查看爬虫运行情况      
10.110.181.40:6800/jobs

终止爬虫
```shell
curl http://localhost:6800/cancel.json -d project=scrapy_learning -d job=ede3c8dc7c7611e9b28e0050568b7b68
```

列出工程
```shell
curl http://localhost:6800/listprojects.json
```

列出爬虫
```shell
curl http://localhost:6800/listspiders.json?project=scrapy_learning
```

列出job
```shell
curl http://localhost:6800/listjobs.json?project=scrapy_learning
```

删除项目
```shell
curl http://localhost:6800/delproject.json?project=scrapy_learning
```

### Scrapyd API           
```shell
pip install python-scrapyd-api
```
使用介绍：      
https://pypi.org/project/python-scrapyd-api/            
https://python-scrapyd-api.readthedocs.io/en/latest/usage.html#quick-usage           

### 参考资料    
[PyCharm中的scrapy安装教程](https://www.cnblogs.com/xiaoli2018/p/4566639.html)    
[利用scrapy和MongoDB来开发一个爬虫](https://www.cnblogs.com/tianyake/p/5518200.html)    
[零基础写python爬虫之使用scrapy框架编写爬虫](https://www.jb51.net/article/57183.htm)    
[scrapy框架爬取豆瓣网电影排行榜](https://blog.csdn.net/weixin_41048363/article/details/79681232)    
