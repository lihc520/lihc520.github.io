# Kubenetes使用Hlem安装nfs为默认存储


### 1、部署 Helm 客户端

Helm客户端下载地址：https://github.com/helm/helm/releases

解压移动到/usr/bin/目录即可。

```bash
wget https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz
tar zxvf helm-v3.3.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/
```

### 2、命令补全

```bash
# 安装epel源
yum -y install epel-release
yum -y install bash-completion
helm completion bash > /etc/bash_completion.d/helm

# Alpine 系统

# bash 文档 可以不按照
apk add bash-doc
# bash 自动命令补全
apk add bash-completion
helm completion bash > /usr/share/bash-completion/helm
```



### 3、Helm常用命令

| **命令**   | **描述**                                                     |
| ---------- | ------------------------------------------------------------ |
| create     | 创建一个chart并指定名字                                      |
| dependency | 管理chart依赖                                                |
| get        | 下载一个release。可用子命令：all、hooks、manifest、notes、values |
| history    | 获取release历史                                              |
| install    | 安装一个chart                                                |
| list       | 列出release                                                  |
| package    | 将chart目录打包到chart存档文件中                             |
| pull       | 从远程仓库中下载chart并解压到本地 # helm pull stable/mysql –untar |
| repo       | 添加，列出，移除，更新和索引chart仓库。可用子命令：add、index、list、remove、update |
| rollback   | 从之前版本回滚                                               |
| search     | 根据关键字搜索chart。可用子命令：hub、repo                   |
| show       | 查看chart详细信息。可用子命令：all、chart、readme、values    |
| status     | 显示已命名版本的状态                                         |
| template   | 本地呈现模板                                                 |
| uninstall  | 卸载一个release                                              |
| upgrade    | 更新一个release                                              |
| version    | 查看helm客户端版本                                           |



### 4、配置国内Chart仓库

- 微软仓库（http://mirror.azure.cn/kubernetes/charts/）这个仓库强烈推荐，基本上官网有的chart这里都有。
- 阿里云仓库（https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts ）
- 官方仓库（https://hub.kubeapps.com/charts/incubator）官方chart仓库，国内使用有点慢。

添加存储库：

```bash
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts 
helm repo update
```

查看配置的存储库：

```bash
helm repo list
helm search repo stable
```



### 5、部署 NFS 服务

#### 5.1、 安装 NFS 与 rpcbind 服务

```bash
#   创建 NFS 存储目录
mkdir -p /data/k8s
#   安装nfs服务
yum -y install nfs-utils rpcbind
#   修改配置文件
echo "/data/NFS *(rw,sync,no_root_squash,no_subtree_check)" > /etc/exports
#   启动服务
systemctl start nfs && systemctl start rpcbind
#   设置开机启动
systemctl enable nfs-server && systemctl enable rpcbind
```

#### 5.2、集群节点安装 nfs

重点全部节点都需要安装 nfs-utils 安装了即可不需要配置，否则节点无法挂载 pv

```
#   安装nfs服务
yum -y install nfs-utils

#		在节点上测试nfs服务器上的可挂载目录
showmount -e 10.168.3.158 ##nfs服务器地址
```

#### 5.3、 使用 helm 安装 nfs-client-provisioner为默认存储

```bash
helm install nfs-client-provisioner \				
  --set storageClass.name=nfs-client \			## 存储类名
  --set storageClass.defaultClass=true \		## 指定为默认存储		
  --set nfs.server=10.168.3.158 \						## nfs服务器地址	
  --set nfs.path=/data/k8s \								## 挂载路径
  stable/nfs-client-provisioner
  
## 部署后查看存储状况
[root@master3 /]# kubectl get sc
NAME                   PROVISIONER                            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client (default)   cluster.local/nfs-client-provisioner   Delete          Immediate           true                   17h
```

#### 

