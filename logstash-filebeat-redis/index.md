# logstash+filebeat+redis


# ELK

### 客户端部署

目前的架构是elasticsearch+logstash+filebeat+kibana+redis

通过filebeat收集数据发送到redis，然后logstash从redis中取数据发送到elasticsearch，最后通过kibana展示。

本文章讲述如何收集客户端日志。

es：10.168.2.44 	端口9200

redis:	10.168.2.205	端口30379



#### 安装logstash

下载安装包： wget https://artifacts.elastic.co/downloads/logstash/logstash-7.13.2-linux-x86_64.tar.gz 

解压：tar zxvf logstash-7.13.2-linux-x86_64.tar.gz  && mv logstash-7.13.2 /usr/local/logstash

**创建logstash.conf文件，添加以下内容**

```sh
input {
   redis {
     host => "10.168.2.205"
     port => "30379"
     db => "2"
     password => "password"
     data_type => "list"
     key => "vpn_log"
   }
}
output {
    if [fields][app] == "system" {
        elasticsearch {
            hosts => ["10.168.2.44:9200"]
            user => "elastic"
            password => "password"
            index => "vpn_syslog-%{+YYYY.MM.dd}"
        }
    }
    if [fields][app] == "vpn" {
        elasticsearch {
            hosts => ["10.168.2.44:9200"]
            user => "elastic"
            password => "password"
            index => "vpn_server-%{+YYYY.MM.dd}"
        }
    }
}

### 注意logstash与filebeat里的配置文件参数要对应
```

#### 安装filebeat

下载安装包： wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.13.2-linux-x86_64.tar.gz

解压：tar zxvf filebeat-7.13.2-linux-x86_64.tar.gz && mv filebeat-7.13.2-linux-x86_64 /usr/local/filebeat

修改filebeat.yml配置文件：

```sh
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/messages  ##位置
  fields:
    type: "vpn_syslog"   ## 类型
    app: "syslog"				##类型
- type: log
  enabled: true
  paths:
    - /root/vpnserver/server_log/*.log
  fields:
    type: "vpn_server"
    app: "vpn"
output.redis:
  enabled: true
  hosts: ["10.168.2.205:30379"]
  password: "password"
  key: "vpn_log"	
  db: 2
  timeout: 10
  
  ## 注意参数和上面的logstash一样
```

#### 启动

filebeat：

```sh
 cd /usr/local/filebeat
 ./filebeat -e -c filebeat.yml >>logs/filebeat.log  #前台启动，先前台启动一遍看有没有报错，没有的话就关闭进行后台运行
 nohup ./filebeat -e -c filebeat.yml >>logs/filebeat.log >/dev/null 2>&1 &  #后台启动
```



logstash：

```sh
/usr/local/logstash/bin/logstash -f /usr/local/logstash/config/nginx.conf #前台启动，先前台启动一遍看有没有报错，没有的话就关闭进行后台运行
cd /usr/local/logstash/bin && nohup ./logstash -f /usr/local/logstash/config/nginx.conf >/dev/null 2>&1 &  #后台启动
```

 查看启动状态

```sh
ps -ef | grep logstash
ps -ef | grep filebeat
```


