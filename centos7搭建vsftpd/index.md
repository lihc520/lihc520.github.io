# centos7搭建vsftpd


# centos7搭建 Vsftpd 服务。

###### 安装服务端程序

```
[root@localhost ~]# yum install vsftpd
Loaded plugins: langpacks, product-id, subscription-manager
………………省略部分输出信息………………
================================================================================
Installing:
vsftpd x86_64 3.0.2-9.el7 rhel 166 k
Transaction Summary
================================================================================
Install 1 Package
vsftpd.x86_64 0:3.0.2-9.el7
Complete!

[root@linuxprobe ~]# yum install ftp
Loaded plugins: langpacks, product-id, subscription-manager
………………省略部分输出信息………………
Install 1 Package
ftp.x86_64 0:0.17-66.el7
Complete!
```

###### 关闭本机防火墙和禁用selinux

```
[root@localhost ~]# systemctl stop firewalld && systemctl disable firewalld
[root@localhost ~]# setenforce 0
[root@localhost ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

###### 配置 FTP 服务

```
# 程序的主配置文件为/etc/vsftpd/vsftpd.conf
[root@localhost ~]# cat /etc/vsftpd/vsftpd.conf
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

###### 配置文件

CentOS7 中的配置文件位置

```
主配置文件：/etc/vsftpd/vsftpd.conf
配置文件目录：/etc/vsftpd/*.conf
服务启动脚本：/etc/rc.d/init.d/vsftpd
用户认证配置文件：/etc/pam.d/vsftpd
```

###### 共享目录

```
匿名用户(映射为ftp用户)共享资源位置：/var/ftp
系统用户通过ftp访问的资源的位置：用户自己的家目录
虚拟用户通过ftp访问的资源的位置：给虚拟用户指定的映射成为的系统用户的家目录
```

###### 通用基础配置

```
listen=[YES|NO]         #是否以独立运行的方式监听服务
listen_address=IP地址   #设置要监听的 IP 地址
listen_port=21        #设置 FTP 服务的监听端口
download_enable＝[YES|NO] #是否允许下载文件
max_clients=0   #最大客户端连接数，0 为不限制
max_per_ip=0   #同一 IP 地址的最大连接数，0 为不限制
chown_uploads=[YES|NO] #是否允许改变上传文件的属主
chown_username=whoever #改变上传文件的属主为 whoever
pam_service_name=vsftpd #让 vsftpd 使用 pam 完成用户认证，使用的文件为/etc/pam.d/vsftpd
```

###### 匿名用户的配置

```
anonymous_enable=[YES|NO]      #是否允许匿名用户访问
anon_upload_enable=[YES|NO]    #是否允许匿名用户上传文件
anon_mkdir_write_enable=[YES|NO] #是否允许匿名用户创建目录
anon_other_write_enable=[YES|NO] #是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限）
anon_umask=022       #匿名用户上传文件的 umask 值
anon_root=/var/ftp   #匿名用户的 FTP 根目录
anon_max_rate=0       #匿名用户的最大传输速率（字节/秒），0 为不限制
```

###### 系统用户的配置

```
anonymous_enable=NO    #禁止匿名访问模式
local_enable=[YES|NO]  #是否允许本地用户登录 FTP
write_enable=[YES|NO]  #是否开放本地用户的其他写入权限
local_umask=022        #本地用户上传文件的 umask 值
local_root=/var/ftp    #本地用户的 FTP 根目录
local_max_rate=0      #本地用户最大传输速率（字节/秒），0 为不限制
userlist_enable=[YES|NO] #开启用户作用名单文件功能
userlist_deny=[YES|NO]   #启用禁止用户名单，名单文件为 ftpusers 和/etc/vsftpd/user_list
chroot_local_user=[YES|NO] #是否将用户权限禁锢在 FTP 家目录中，以确保安全
chroot_list_enable=[YES|NO] #禁锢文件中指定的 FTP 本地用户于其家目录中
chroot_list_file=/etc/vsftpd/chroot_list #指定禁锢文件位置，需要和 chroot_list_enable 一同开启
```

###### 日志功能

```
xferlog_enable=[YES|NO]    #是否开启 FTP 日志功能
xferlog_std_format=[YES|NO]    #是否以标准格式保持日志
xferlog_file=/var/log/xferlog  #指定保存日志的文件名称，需要一同开启
```

## vsftpd 认证模式

vsftpd 作为更加安全的文件传输的服务程序，允许用户以三种认证模式登录到 FTP 服务器上。

- 匿名开放模式

- - 匿名开放模式是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到FTP服务器。这种模式一般用来访问不重要的公开文件，在生产环境中尽量不要存放重要文件，不建议在生产环境中如此行事。

- 本地用户模式

- - 本地用户模式是通过Linux系统本地的账户密码信息进行认证的模式，相较于匿名开放模式更安全，而且配置起来相对简单。但是如果被黑客破解了账户的信息，就可以畅通无阻地登录FTP服务器，从而完全控制整台服务器。

- 虚拟用户模式

- - 虚拟用户模式是这三种模式中最安全的一种认证模式，它需要为FTP服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供FTP服务程序进行认证使用。这样，即使黑客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响。

    

### 匿名访问模式

vsftpd 服务程序默认开启了匿名开放模式，我们需要做的就是开放匿名用户的上传、下载文件的权限，以及让匿名用户创建、删除、更名文件的权限。

```
# 匿名访问模式主配置文件
[root@localhost ~]# vim /etc/vsftpd/vsftpd.conf
anonymous_enable=YES
anon_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES

local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
# 重启服务
[root@localhost ~]# systemctl restart vsftpd

# 服务程序加入到开机启动项中，以保证服务器在重启后依然能够正常提供传输服务
[root@localhost ~]# systemctl enable vsftpd
ln -s '/usr/lib/systemd/system/vsftpd.service' '/etc/systemd/system/multi-user.target.wants/vsftpd.service
# 在vsftpd服务程序的匿名开放认证模式下，其账户统一为anonymous，密码为空
# 连接到FTP服务器后，默认访问的是/var/ftp目录，我们可以在其中进行创建、删除等操作
[root@localhost ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
550 Permission denied.
# 系统显示拒绝创建目录，这是为什么呢？
# 查看该目录的权限得知，只有root管理员才有写入权限，开放ftp用户权限(该账户在系统中已经存在)
[root@localhost ~]# ls -ld /var/ftp/pub
drwxr-xr-x. 3 root root 16 Jul 13 14:38 /var/ftp/pub

[root@localhost ~]# chown -Rf ftp /var/ftp/pub
[root@localhost ~]# ls -ld /var/ftp/pub
drwxr-xr-x. 3 ftp root 16 Jul 13 14:38 /var/ftp/pub

[root@linuxprobe ~]# ftp 192.168.10.10
………………省略部分输出信息………………
ftp> mkdir files
257 "/pub/files" created
ftp> rename files database
350 Ready for RNTO.
250 Rename successful.
ftp> exit
221 Goodbye.
```

### 本地用户模式

如果之前用的是匿名开放模式，现在就可以将它关了，然后开启本地用户模式。

```
# 本地用户模式主配置文件
[root@localhost ~]# vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO

local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
connect_from_port_20=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
xferlog_enable=YES
xferlog_std_format=YES
# 同样需要重启服务和开机自启动
[root@localhost ~]# systemctl restart vsftpd

# 服务程序加入到开机启动项中，以保证服务器在重启后依然能够正常提供传输服务
[root@localhost ~]# systemctl enable vsftpd
ln -s '/usr/lib/systemd/system/vsftpd.service' '/etc/systemd/system/multi-user.target.wants/vsftpd.service
# 现在已经完全可以本地用户的身份登录FTP服务器了，但是使用root无法登陆
[root@localhost ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): root
530 Permission denied.
Login failed.
ftp>

# 这是因为，为了系统的安全，默认禁止root等用户登录FTP服务被系统拒绝访问
# 因为vsftpd服务程序所在的目录中，默认存放着两个名为用户名单的文件，ftpusers和user_list
# 在ftpusers和user_list两个用户文件中将root用户删除就可以登录了
[root@localhost ~]# cat /etc/vsftpd/user_list
root
bin
daemon

[root@localhost ~]# cat /etc/vsftpd/ftpusers
root
bin
daemon
# 在采用本地用户模式登录FTP服务器后，默认访问的是该用户的家目录，因此不存在写入权限不足的情况
# 如果不关闭SELinux，则需要再次开启SELinux域中对FTP服务的允许策略
[root@localhost ~]# setsebool -P ftpd_full_access=on

# 即可以使用系统用户进行FTP服务的登录了
[root@localhost ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): escape
331 Please specify the password.
Password:此处输入该用户的密码
230 Login successful.
Remote system type is UNIX.
```

### 虚拟用户模式(文本文件)

认证模式：vsftpd + pam + file

- #### 第一步：创建用于进行 FTP 认证的用户数据库文件

- - 这里使用文本文件进行用户认证
  - 数据库文件中奇数行为账户名，偶数行为密码

```
# 编辑虚拟用户文件
[root@localhost ~]# cd /etc/vsftpd/
[root@localhost vsftpd]# vim vuser.list
aimo1
aimo@2021
aimo2
aimo@2021
# 明文信息既不安全，也不符合让vsftpd服务程序直接加载的格式
# 因此需要使用db_load命令用哈希算法将原始的明文信息文件转换成数据库文件
# 降低数据库文件的权限，然后再把原始的明文信息文件删除
[root@localhost vsftpd]# db_load -T -t hash -f vuser.list vuser.db
[root@localhost vsftpd]# file vuser.db
vuser.db: Berkeley DB (Hash, version 9, native byte-order)

[root@localhost vsftpd]# chmod 600 vuser.db
```

- #### 第二步：创建 vsftpd 服务程序用于存储文件的根目录以及虚拟用户映射的系统本地用户

- - FTP服务用于存储文件的根目录指的是，当虚拟用户登录后所访问的默认位置
  - 可以把这个系统本地用户的家目录设置为/var目录并设置不允许登录FTP服务器

```
[root@localhost ~]# useradd -d /var/ftproot -s /sbin/nologin virtual

[root@localhost ~]# ls -ld /var/ftproot/
drwx------. 3 virtual virtual 74 Jul 14 17:50 /var/ftproot/

[root@localhost ~]# chmod -Rf 755 /var/ftproot/
```

- #### 第三步：建立支持虚拟用户的 PAM 认证文件

- - PAM是一种认证机制，通过一些动态链接库和统一的API把系统提供的服务与认证方式分开
  - PAM是可插拔认证模块，使得系统管理员可以根据需求灵活调整服务程序的不同认证方式

```
# 新建一个用于虚拟用户认证的PAM文件vsftpd.vu
# PAM文件内的db=参数为使用db_load命令生成的账户密码数据库文件的路径，但不用写数据库文件的后缀

[root@localhost ~]# vim /etc/pam.d/vsftpd.vu
auth       required     pam_userdb.so    db=/etc/vsftpd/vuser
account    required     pam_userdb.so    db=/etc/vsftpd/vuser
```

- #### 第四步：在 vsftpd.conf 文件中添加支持配置

- - 在vsftpd服务程序的主配置文件中默认就带有参数pam_service_name=vsftpd
  - 表示登录FTP服务器时是根据/etc/pam.d/vsftpd文件进行安全认证的

```
# 我们要做的就是把vsftpd主配置文件中原有的PAM认证文件vsftpd修改为新建的vsftpd.vu文件即可
[root@localhost ~]# vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
pam_service_name=vsftpd.vu
user_config_dir=/etc/vsftpd/vusers_dir

local_enable=YES
guest_enable=YES
guest_username=virtual
allow_writeable_chroot=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
userlist_enable=YES
tcp_wrappers=YES
```

- #### 第五步：为虚拟用户设置不同的权限

- - 只需新建一个目录，在里面分别创建两个以aimo1和aimo2命名的文件
  - 在每个文件中，对用户分别进行配置，达到管理不用用户权限的效果

```
[root@localhost ~]# mkdir /etc/vsftpd/vusers_dir/
[root@localhost ~]# cd /etc/vsftpd/vusers_dir/

[root@localhost vusers_dir]# touch aimo1 aimo2

[root@localhost vusers_dir]# vim aimo1
local_root=/usr/local/vsftpd/aimo1		#设置虚拟用户目录
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES

[root@localhost vusers_dir]# vim aimo2
local_root=/usr/local/vsftpd/aimo2
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
```

- #### 第六步：使用虚拟 FTP 用户访问测试

```
[root@localhost ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): lisi
331 Please specify the password.
Password:此处输入虚拟用户的密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> mkdir files
550 Permission denied.
ftp> exit
221 Goodbye.
[root@localhost ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): zhangsan
331 Please specify the password.
Password:此处输入虚拟用户的密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> mkdir files
257 "/files" created
ftp> rename files database
350 Ready for RNTO.
250 Rename successful.
ftp> rmdir database
250 Remove directory operation successful.
ftp> exit
221 Goodbye.
```

最后重启vsftpd服务

```
[root@localhost ~]# systemctl restart vsftpd
```


