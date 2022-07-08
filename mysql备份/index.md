# Mysql备份


# [MySql数据库备份的几种方式]

### **mysqldump工具备份**

备份整个数据库

```
$> mysqldump -u root -h host -p dbname > backdb.sql

```

备份数据库中的某个表

```
$> mysqldump -u root -h host -p dbname tbname1, tbname2 > backdb.sql

```

备份多个数据库

```
$> mysqldump -u root -h host -p --databases dbname1, dbname2 > backdb.sql

```

备份系统中所有数据库

```
$> mysqldump -u root -h host -p --all-databases > backdb.sql

```

### **直接复制整个数据库目录(对于InnoDB存储引擎不适用)备份**

windowns: installpath/mysql/data

linux: /var/lib/mysql

在复制前需要先执行如下命令：

```
MYSQL> LOCK TABLES;
# 在复制过程中允许客户继续查询表，
MYSQL> FLUSH TABLES;
# 将激活的索引页写入硬盘。

```

### **mysqlhotcopy工具备份**

备份数据库或表最快的途径，只能运行在数据库目录所在的机器上，并且只能备份MyISAM类型的表。

要使用该备份方法必须可以访问备份的表文件。

```
$> mysqlhotcopy -u root -p dbname /path/to/new_directory;
#将数据库复制到new_directory目录。

```

### **mysql命令导入sql文件还原**

```
$> mysql -u root -p [dbname] < backup.sql
# 执行前需要先创建dbname数据库，如果backup.sql是mysqldump创建的备份文件则执行是不需要dbname。
MYSQL> source backup.sql;
# 执行source命令前需要先选择数据库。

```

### **直接复制数据库目录还原**

**注： 该方式必须确保原数据库和待还原的数据库主版本号一致，并且只适用于MyISAM引擎的表。**

1. 关闭mysql服务。
2. 将备份的文件或目录覆盖mysql的data目录。
3. 启动mysql服务。
4. 对于linux系统，复制完文件后需要将文件的用户和组更改为mysql运行的用户和组。

### **mysqlhotcopy快速恢复**

停止mysql服务，将备份数据库文件复制到存放数据的位置（mysql的data文件夹），重先启动mysql服务即可(可能需要指定数据库文件的所有者)。

```
$> cp -R /usr/backup/test /usr/local/mysql/data
# 如果恢复的数据库已经存在，则使用DROP语句删除已经存在的数据库之后，恢复才能成功，还需要保证数据库版本兼容。

```

### **相同版本数据库之间迁移**

```
$> mysqldump -h www.abc.com -uroot -p password dbname |
$> mysqldump -h www.bcd.com -uroot -p password
# 将服务器www.abc.com的数据库dbname迁移到服务器www.bcd.com的相同版本数据库上。

```

### **不同版本的mysql数据库之间的迁移**

备份原数据库。

卸载原数据库。

安装新数据库。

在新数据库中还原备份的数据库数据。

数据库用户访问信息需要备份mysql数据库。

默认字符集问题，MySQL4.x中使用latin1作为默认字符集，mysql5.x使用utf8作为默认字符集。如果有中文数据需要对默认字符集进行更改。

不同数据库之间的迁移

MyODBC工具实现MySQL和SQL Server之间的迁移。

MySQL Migration Toolkit工具。

### **表的导出和导入**

SELECT ...... INTO OUTFILE 导出文本文件,该方法只能导出到数据库服务器上，并且导出文件不能已存在。

```
MYSQL> SELECT ...... INTO OUTFILE filename [OPTIONS]
MYSQL> SELECT * FROM test.person INTO OUTFILE "C:\person0.txt";
# 将表person里的数据导入为文本文件person0.txt。

```

mysqldump文件导出文本文件(和INTO OUTFILE不一样的是该方法所有的选项不需要添加引号)

```
$> mysqldump -T path -u root -p dbname [tables] [OPTIONS]
# -T参数表明导出文本文件。path导出数据的目录。
$> mysqldump -T C:\test person -u root -p
# 将test表中的person表导出到文本文件。执行成功后test目录下会有两个文件，person.sql和person.txt

```

mysql命令导出文本文件

```
MYSQL> mysql -u root -p --execute="SELECT * FROM person;" test > C:\person3.txt;
# 将test数据库中的person表数据导出到person3.txt文本文件中。--vartical参数可以将一行分为多行显示。
MYSQL> mysql -u root -p --vartical --execute="SELECT * FROM person;" test > C:\person3.txt;
# --html将表导出为html文件，--xml文件将表导出为xml文件

```

LOAD DATA INFILE导入文本文件

```
MYSQL> LOAD DATA INFILE 'filename.txt' INTO TABLE tablename [OPTIONS] [IGNORE number LINES];
# [IGNORE number LINES]表示忽略行数
MYSQL> LOAD DATA INFILE 'C:\person0.txt' INTO TABLE test.person;

```

mysqlimport导入文本文件

```
$> mysqlimport -u root -p dbname filename.txt [OPSTONS]
# 导入的表名有文件名决定，导入数据之前表必须存在
$> mysqlimport -uroot -p test C:\backup\person.txt
# 将数据导入到test数据库的person表中。

```

### **使用mysqlbinlog恢复数据**

```
$> mysqlbinlog [option] filename | mysql -u user -p password
# filename为二进制日志文件，
$> mysqlbinlog --stop-date="2013-03-30 15:27:47" D:\MySQL\log\binlog\binlog.000008 | mysql -u root -p password
# 根据日志文件binlog.000008将数据恢复到2013-03-30 15:27:47以前的操作。

```

### **启动二进制日志**

```
log-bin = path/filename	#日志文件存储目录和文件名
expire_log_days = 10	#日志自动删除时间
max_binlog_size = 100M	# 日志文件最大大小

```

### **查看二进制日志**

```
MYSQL> SHOW VARIABLES LIKE 'log_%';
MYSQL> SHOW BINARY LOGS;
$> mysqlbinlog filename
# filename为二进制日志文件名。

```

### **删除二进制日志**

```
MYSQL> RESET MASTER; #删除所有二进制日志MYSQL> PURGE {MASTER | BINARY} LOGS TO 'log_name';	#删除文件编号小于log_name编号的文件MYSQL> PURGE {MASTER | BINARY} LOGS BEFORE 'date';	#删除指定日期以前的文件
```

### **暂时停止二进制日志（不需要重启mysql服务）**

```
MYSQL> SET sql_log_bin = {0|1}	#暂停或启动二进制日志。
```

```bash
一. mysqldump命令备份数据
在MySQL中提供了命令行导出数据库数据以及文件的一种方便的工具mysqldump,我们可以通过命令行直接实现数据库内容的导出dump,首先我们简单了解一下mysqldump命令用法:

#MySQLdump常用
mysqldump -u root -p --databases 数据库1 数据库2 > xxx.sql
二. mysqldump常用操作示例
1.备份全部数据库的数据和结构
mysqldump -uroot -p123456 -A > /data/mysqlDump/mydb.sql
2.备份全部数据库的结构（加 -d 参数）
mysqldump -uroot -p123456 -A -d > /data/mysqlDump/mydb.sql
3.备份全部数据库的数据(加 -t 参数)
mysqldump -uroot -p123456 -A -t > /data/mysqlDump/mydb.sql
4.备份单个数据库的数据和结构(,数据库名mydb)
mysqldump -uroot-p123456 mydb > /data/mysqlDump/mydb.sql
5. 备份单个数据库的结构
mysqldump -uroot -p123456 mydb -d > /data/mysqlDump/mydb.sql
6. 备份单个数据库的数据
mysqldump -uroot -p123456 mydb -t > /data/mysqlDump/mydb.sql
7. 备份多个表的数据和结构（数据，结构的单独备份方法与上同）
mysqldump -uroot -p123456 mydb t1 t2 > /data/mysqlDump/mydb.sql
8. 一次备份多个数据库
mysqldump -uroot -p123456 --databases db1 db2 > /data/mysqlDump/mydb.sql
三. 还原 MySQL 备份内容
有两种方式还原，第一种是在 MySQL 命令行中，第二种是使用 SHELL 行完成还原

1.在系统命令行中，输入如下实现还原：
mysql -uroot -p123456 < /data/mysqlDump/mydb.sql
2.在登录进入mysql系统中,通过source指令找到对应系统中的文件进行还原：
mysql> source /data/mysqlDump/mydb.sql
```
