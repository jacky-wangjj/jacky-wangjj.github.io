---
layout: post
title: Hive表支持中文设置
date: 2019-05-07
tags: 大数据
---  

### Hive表支持中文设置
修改hive的元数据库相关表的属性，元数据库一般是存储在mysql中的，执行如下SQL语句，然后可以在hive中创建带中文的表。    
```sql
alter table  TBLS  modify column TBL_NAME  varchar(1000) character set utf8;
alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;
alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
alter table PARTITION_PARAMS  modify column PARAM_VALUE varchar(4000) character set utf8;
alter table PARTITION_KEYS  modify column PKEY_COMMENT varchar(4000) character set utf8;
alter table  INDEX_PARAMS  modify column PARAM_VALUE  varchar(4000) character set utf8;
```    
测试如下
```sql
create table if not exists studentno (stuno string comment '学号', stuname1 string comment '姓名') comment '学生信息表';
```    

hive表中字段名支持中文，中文字段使用反引号```括起来。    
```sql
create table student(`学号` string comment '学号', `姓名` string comment '姓名') comment '学生信息表';
insert into student values('一号','王石');
+-------------+-------------+--+
| student.学号  | student.姓名  |
+-------------+-------------+--+
| 一号          | 王石          |
+-------------+-------------+--+
```
### 参考资料
[hive官网相关说明](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateTable)
Table names and column names are case insensitive but SerDe and property names are case sensitive.
**In Hive 0.12 and earlier**, only alphanumeric and underscore characters are allowed in table and column names.
**In Hive 0.13 and later, column names can contain any Unicode character (see HIVE-6013)**, however, dot (.) and colon (:) yield errors on querying, so they are disallowed in Hive 1.2.0 (see HIVE-10120). Any column name that is specified within backticks (\`) is treated literally. Within a backtick string, use double backticks (\`\`) to represent a backtick character. **Backtick quotation also enables the use of reserved keywords for table and column identifiers.**
To revert to pre-0.13.0 behavior and restrict column names to alphanumeric and underscore characters, set the configuration property [hive.support.quoted.identifiers](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.support.quoted.identifiers) to none. In this configuration, backticked names are interpreted as regular expressions. For details, see [Supporting Quoted Identifiers in Column Names]().
**Table and column comments are string literals (single-quoted).**     

[Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-date)    

[Hive Data Definition Language](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)    
