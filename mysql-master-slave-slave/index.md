# MySql:master:slave:slave


| Hosts               | CPU  | MEM  | HD   | OS          |
| ------------------- | ---- | ---- | ---- | ----------- |
| 192.168.1.102  db01 | 8    | 16G  | 100G | Ubuntu20.04 |
| 192.168.1.105  db02 | 8    | 16G  | 100G | Ubuntu20.04 |
| 192.168.1.108  db03 | 8    | 16G  | 100G | Ubuntu20.04 |

安装前修改两台服务器的hosts

```sh
# 三臺台服务器都要操作
sudo vim /etc/hosts
192.168.1.102 db01
192.168.1.105 db02
192.168.1.108 db03
```

創建MySQL存儲位置及日誌位置

```sh
sudo mkdir -p /data/mysql && sudo mkdir -p /data/logs/mysql
sudo chmod -R mysql:mysql /data/mysql && sudo chmod -R mysql:mysql /data/logs/mysql
```



### MySql5.7安装(三臺服務器進行同樣操作)

```shell
##下载所需安装包
cd /opt
sudo wget http://mirrors.ustc.edu.cn/mysql-ftp/Downloads/MySQL-5.7/mysql-server_5.7.34-1ubuntu18.04_amd64.deb-bundle.tar
sudo tar -xvf mysql-server_5.7.34-1ubuntu18.04_amd64.deb-bundle.tar  #解压安装包

sudo wget http://archive.ubuntu.com/ubuntu/pool/main/m/mecab/libmecab2_0.996-10build1_amd64.deb #下载依赖包

```

db1上操作

在安装包所在的目录下执行（安装顺序不可更改）

##### 1、安装`mysql-common_5.7.34-1ubuntu18.04_amd64.deb`

```css
sudo apt install libtinfo5 -y
sudo dpkg -i mysql-common_5.7.34-1ubuntu18.04_amd64.deb

```



##### 2、安装`mysql-community-client_5.7.34-1ubuntu18.04_amd64.deb`

```css
sudo dpkg -i mysql-community-client_5.7.34-1ubuntu18.04_amd64.deb
```



##### 3、安装`mysql-client_5.7.34-1ubuntu18.04_amd64.deb`

```css
sudo dpkg -i mysql-client_5.7.34-1ubuntu18.04_amd64.deb
```



##### 4、安装 `mysql-community-server_5.7.29-1ubuntu18.04_amd64.deb`

安装过程中会提示缺少依赖包libmecab2：
 所以这次先安装依赖包`libmecab2`：

```css
sudo dpkg -i libmecab2_0.996-10build1_amd64.deb
```

然后安装`mysql-community-server_5.7.34-1ubuntu18.04_amd64.deb`

```css
sudo dpkg -i mysql-community-server_5.7.34-1ubuntu18.04_amd64.deb
```



安装过程中会要求在粉色的大页面上输入密码，输入两次即可，至此安装完成。

##### 5、修改配置文件

```sh
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

#将内容修改成如下
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /data/mysql
log-error       = /data/logs/mysql/error.log

sql_mode =

user = mysql
bind-address = 0.0.0.0
port = 3307 

log-bin = mysql-bin
server-id = 1
binlog-ignore-db = mysql,information_schema,performance_schema
auto-increment-offset = 3
auto-increment-increment = 3

slave_parallel_type = 'logical_clock'
slave_parallel_workers = 8

relay_log = relay-bin
log-slave-updates = ON

init-connect = 'SET NAMES utf8mb4'
character-set-server = utf8mb4

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

default_storage_engine = InnoDB
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120

bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1

interactive_timeout = 28800
wait_timeout = 28800

explicit_defaults_for_timestamp = true
```

修改MySQL存儲位置後需要修改/etc/apparmor.d/usr.sbin.mysqld 

```sh
sudo vim /etc/apparmor.d/usr.sbin.mysqld# Allow data dir access  /data/mysql/ r,  /data/mysql/** rwk,  # Allow log file access  /data/logs/mysql/ r,  /data/logs/mysql/** rwk,  # 重啟apparmorsudo systemctl restart apparmor
```



###### 以上操作在DB2和DB3上面重复操作，只需修改配置文件中的server-id = 2



启动mysql：`sudo service mysql start`
停止mysql：`sudo service mysql stop`
重启mysql：`sudo service mysql restart`

加入开机启动：`sudo systemctl enable mysql`



### MySql主備模式

```sh
# db01
#进入数据库
sudo mysql -u root -p 

#创建同步用户
mysql> grant all on *.* to replication@'%' identified by 'password';
mysql> flush privileges;

#查看记录master status
mysql> show master status;
+------------------+----------+--------------+---------------------------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB                            | Executed_Gtid_Set |
+------------------+----------+--------------+---------------------------------------------+-------------------+
| mysql-bin.000003 |     1288 |              | mysql,information_schema,performance_schema |                   |
+------------------+----------+--------------+---------------------------------------------+-------------------+
1 row in set (0.01 sec)
```



```sh
# db02和db03都進行一下操作
#进入数据库
sudo mysql -u root -p 

#创建同步用户
mysql> grant all on *.* to replication@'%' identified by 'password';
mysql> flush privileges;

#查看记录master status
mysql> show master status;
+------------------+----------+--------------+---------------------------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB                            | Executed_Gtid_Set |
+------------------+----------+--------------+---------------------------------------------+-------------------+
| mysql-bin.000004 |     1288 |              | mysql,information_schema,performance_schema |                   |
+------------------+----------+--------------+---------------------------------------------+-------------------+
1 row in set (0.00 sec)
```



进行同步

```sh
##db02
#设置主服务器的信息
mysql> change master to master_host='db01',master_port=3307,master_user='replication', master_password='password', master_log_file='mysql-bin.000004', master_log_pos=1288;

# 启动同步
mysql> start slave; 

#查看同步状态，要两个yes才成功
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: db1
                  Master_User: replication
                  Master_Port: 3307
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 1288
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 484
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1288
              Relay_Log_Space: 685
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: f2770e08-e2fd-11eb-a6f7-005056b9aeb1
             Master_Info_File: /data/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```



```sh
#db03
#设置主服务器的信息
mysql> change master to master_host='db03',master_port=3307,master_user='replication', master_password='password', master_log_file='mysql-bin.000004', master_log_pos=1288;

# 启动同步
mysql> start slave; 

#查看同步状态，要两个yes才成功
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: db1
                  Master_User: replication
                  Master_Port: 3307
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 1288
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 484
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1288
              Relay_Log_Space: 685
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: f2770e08-e2fd-11eb-a6f7-005056b9aeb1
             Master_Info_File: /data/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

mysql主備同步完成；














