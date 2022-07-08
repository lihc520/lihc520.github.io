# NTP服务器与客户端时间同步


### 操作场景

ntpd（Network Time Protocol daemon）是 Linux 操作系统的一个守护进程，用于校正本地系统与时钟源服务器之前的时间，完整的实现了 NTP 协议。ntpd 与 ntpdate 的区别是 ntpd 是步进式的逐渐校正时间，不会出现时间跳变，而 ntpdate 是断点更新。

### 一、公共 NTP 网络时间服务器

#### 1.1、阿里云

- 阿里云: https://help.aliyun.com/document_detail/92704.html

```bash
server ntp1.aliyun.com
server ntp2.aliyun.com
server ntp3.aliyun.com
server ntp4.aliyun.com
server ntp5.aliyun.com
server ntp6.aliyun.com
server ntp7.aliyun.comCOPY
```

#### 1.2、腾讯云

- 腾讯云: https://cloud.tencent.com/document/product/213/30392

```bash
server time1.cloud.tencent.com
server time2.cloud.tencent.com
server time3.cloud.tencent.com
server time4.cloud.tencent.com
server time5.cloud.tencent.comCOPY
```

### 二、部署 NTP 服务

#### 2.1、安装 NTP 服务

- 服务端和客户端机器都需要安装

```bash
#   1、安装 ntp 服务
yum install ntp -y
2、设置 ntp 开机启动
systemctl enable ntpd.serviceCOPY
```

#### 2.2、配置服务端

- 需要修改 2 个地方

```bash
#   1、编辑配置文件
vim /etc/ntp.conf
#   2、修改 restrict 这里修改成网段
restrict 10.168.2.0 mask 255.255.255.0
#   3、修改 server 默认的全部删除 替换成以下内容
server ntp1.aliyun.com
server ntp2.aliyun.com
server ntp3.aliyun.com
server ntp4.aliyun.com
server ntp5.aliyun.com
server ntp6.aliyun.com
server ntp7.aliyun.com
```

#### 2.3、部署客户端

```bash
#   1、编辑配置文件
vim /etc/ntp.conf
#   2、找到 server xxxx 全部删除，修改成 ntp 服务端地址
server 10.168.2.0
```

#### 2.4、启动 NTP 服务

```bash
#   1、服务端和客户端都需要执行
systemctl restart ntpd
#   2、检查 ntpd 状态
netstat -nupl
#   3、查看 ntpd 状态是否正常
service ntpd statusCOPY
```

