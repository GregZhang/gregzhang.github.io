---
title: pt-online-schema-change使用
date: 2017-03-20 13:29:03
tags:
  - pt-online-schema-change
categories: 
  - 运维
---
# 功能介绍：
## 注意:
功能为在`alter`操作更改表结构的时候不用锁定表，也就是说执行`alter`的时候不会阻塞写和读取操作，注意执行这个工具的时候必须做好备份，操作之前最好详细读一下官方文档。
## 工作原理:
是创建一个和你要执行`alter`操作的表一样的空表结构，执行表结构修改，然后从原表中copy原始数据到表结构修改后的表，当数据copy完成以后就会将原表移走，用新表代替原表，默认动作是将原表`drop`掉。在copy数据的过程中，任何在原表的更新操作都会更新到新表，因为这个工具在会在原表上创建触发器，触发器会将在原表上更新的内容更新到新表。如果表中已经定义了触发器这个工具就不能工作了。
# 用法介绍：
```
pt-online-schema-change [OPTIONS] DSN
```
## `options`
可以自行查看help，DNS为你要操作的数据库和表。
这里有两个参数需要介绍一下：
## `--dry-run`
这个参数不建立触发器，不拷贝数据，也不会替换原表。只是创建和更改新表。
## `--execute`
这个参数的作用和前面工作原理的介绍的一样，会建立触发器，来保证最新变更的数据会影响至新表。注意：如果不加这个参数，这个工具会在执行一些检查后退出。这一举措是为了让使用这充分了解了这个工具的原理，同时阅读了官方文档。

# 使用示例：
在线更改表的的引擎，这个尤其在整理innodb表的时候非常有用，示例如下：
```
pt-online-schema-change --user=root --password=XXxx 
    --host=localhost --lock-wait-time=120 
    --alter="ENGINE=InnoDB" D=test,t=oss_pvinfo2 --execute
```
从下面的日志中可以看出它的执行过程：
```
Altering `test`.`oss_pvinfo2`...
Creating new table...
Created new table test._oss_pvinfo2_new OK.
Altering new table...
Altered `test`.`_oss_pvinfo2_new` OK.
Creating triggers...
Created triggers OK.
Copying approximately 995696 rows...
Copied rows OK.
Swapping tables...
Swapped original and new tables OK.
Dropping old table...
Dropped old table `test`.`_oss_pvinfo2_old` OK.
Dropping triggers...
Dropped triggers OK.
Successfully altered `test`.`oss_pvinfo2`.
在来一个范例，大表添加字段的，语句如下:
pt-online-schema-change --user=root --password=XXX 
    --host=localhost --lock-wait-time=120 
    --alter="ADD COLUMN domain_id INT" D=test,t=oss_pvinfo2 --execute
```