# Firewalld


### 开启防火墙

`systemctl start firewalld.service`

### 防火墙开机启动

`systemctl enable firewalld.service`

### 关闭防火墙

`systemctl stop firewalld.service`

### 查看防火墙状态

`firewall-cmd --state`

### 查看现有的规则

`iptables -nL`

### 重载防火墙配置

`firewall-cmd --reload`

### 添加单个单端口

**`firewall-cmd --permanent --zone=public --add-port=81/tcp`**

### 添加多个端口

**`firewall-cmd --permanent --zone=public --add-port=8080-8083/tcp`**

### 删除某个端口

**`firewall-cmd --permanent --zone=public --remove-port=81/tcp`**

# 针对某个 IP开放端口

`firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="6379" accept"`

`firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.233" accept"`

### 删除某个IP

`firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.1.51" accept"`

### 针对一个ip段访问

`firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.0/16" accept"firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="9200" accept"`

### 添加操作后别忘了执行重载

`firewall-cmd --reload`
