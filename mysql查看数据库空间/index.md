# Mysql查看数据库空间


```bash
# 查看总大小
mysql> use information_schema;
mysql> select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from TABLES;
```

```bash
# 查看每个库的大小
SELECT
table_schema,SUM(AVG_ROW_LENGTH*TABLE_ROWS+INDEX_LENGTH)/1024/1024 AS total_mb
FROM information_schema.TABLES group by table_schema;
```

```bash
# 选库查看
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from TABLES where table_schema='tdm_uat';
```

```bash
# 查看所有数据库各容量大小
select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
group by table_schema
order by sum(data_length) desc, sum(index_length) desc;
```

```bash
# 查看所有数据库各表容量大小
select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
order by data_length desc, index_length desc;
```

```bash
# 查看指定数据库容量大小
select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
where table_schema='mysql';
```

```bash
# 查看指定数据库各表容量大小
select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
where table_schema='mysql'
order by data_length desc, index_length desc;
```
