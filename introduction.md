## 注意：

- 在参赛者开始比赛前，需要手动登录实例，建立数据库以及表：


```sql
CREATE DATABASE sql_optimization_match;
USE sql_optimization_match;
CREATE TABLE `order` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL,
  `creator` varchar(24) NOT NULL,
  `price` varchar(64) NOT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `status` tinyint(1) not null,
  PRIMARY KEY (`id`)
);

CREATE TABLE `order_item` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL,
  `parent` bigint(20) NOT NULL,
  `status` int not null,
  `type` varchar(12) NOT NULL DEFAULT '0',
  `quantity` int not null default 1,
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
);
```
```
然后向sql_optimization_match导入原始数据，原始数据在data.sql中
** 如果参赛者想要恢复原始数据，请truncate table 之后重新导入数据
###################################
操作指引
###################################

1.进入sql_optimization_match文件夹，有可执行文件match以及update.sql,select.sql两个sql文件。原始的sql文件里存储着待优
化的sql
2.配置DB地址：按照config_template.json文件，新建文件config.json,在config.json文件里填入自己的db配置
3.参赛者将优化后的sql写入两个文件中，例如update_commit.sql,以及select_commit.sql，最后通过match程序提交sql：
        nohup ./match -operator commit -update update_commit.sql -select select_commit.sql &
4.如何获取帮助信息？
        ./match -h
```

```
select.sql

+----+---------+-----------+-------+---------------------+--------+
| id | name    | creator   | price | create_time         | status |
+----+---------+-----------+-------+---------------------+--------+
|  1 | order_1 | creator_1 | 1373  | 2019-09-27 11:21:23 |      1 |
+----+---------+-----------+-------+---------------------+--------+
+-------------+
| order_count |
+-------------+
|        2000 |
+-------------+
2000订单量
SELECT *
FROM   `order` o
       INNER JOIN order_item i ON i.parent = o.id
ORDER  BY o.status ASC,
          i.update_time DESC
LIMIT  0, 20

update.sql 

+----+----------------+--------+--------+------+----------+---------------------+
| id | name           | parent | status | type | quantity | update_time         |
+----+----------------+--------+--------+------+----------+---------------------+
|  1 | order_1_item_1 |      1 |      0 | 2    |       12 | 2018-08-28 19:51:35 |
|  2 | order_1_item_2 |      2 |      1 | 1    |       12 | 2019-10-24 09:21:22 |
|  3 | order_1_item_3 |      3 |      1 | 5    |       16 | 2018-11-25 06:52:31 |
|  4 | order_1_item_4 |      4 |      0 | 5    |       10 | 2017-11-04 01:56:35 |
|  5 | order_1_item_5 |      5 |      1 | 3    |       15 | 2020-04-03 05:42:15 |
+----+----------------+--------+--------+------+----------+---------------------+
+------------------+
| order_item_count |
+------------------+
|           499760 |
+------------------+
40w:订单包含物品量
          update `order` set
create_time = now()
where id in (
    select parent from order_item where type = 2
)
```