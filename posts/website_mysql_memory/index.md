# mysql内存占用高



<!--more-->

修改 MySQL 配置文件  `my.cnf`，找到  `[mysqld]`  下添加如下内容：
```txt
[mysqld]  
// 此处省略其他配置，添加如下内容  
table_open_cache=200  
table_definition_cache=400  
performance_schema_max_table_instances=400  
performance_schema=off
```
保存然后重启 MySQL，OK！内存已经降到 10%+ 了。

重启mysql命令:
```liunx
systemctl restart mysqld
```
各个配置项的具体用途：

table_open_cache:
	高速缓存的大小，每当访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，下次查询该表时首先从高速缓存区查询，如果表在缓存中则直接从缓存查询，从而大幅提高查询速度。

table_definition_cache:
	定义了内存中可打开的表结构数量。

performance_schema_max_table_instances:
	检测的表对象的最大数目。

performance_schema:
主要用来收集 MySQL 性能参数，启用 performance_schema 之后，server 会持续不间断地监测。【罪魁祸首】

通过调整前面 3 个配置项的值，占用内存均有 1~3% 程度的降低，罪魁祸首便是  `performance_schema`，将其设置为 off 之后，内存直接降低了 20%！

其详细介绍可参考 MySQL 官方文档：[MySQL Performance Schema](https://dev.mysql.com/doc/refman/5.6/en/performance-schema.html)

当然除了上面几个配置项之外，MySQL 仍有许多可以优化的配置项，但是现在既然已经实现了自己的目的，就暂时不进行扩展阅读了
