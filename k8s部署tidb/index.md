# k8s部署TIDB


## k8s部署TIDB

### 服务器建议配置

### 环境

| **组件** | **CPU** | **内存** | **本地存储**     | **网络** | IP           |
| :------- | :------ | :------- | :--------------- | :------- | ------------ |
| Master1  | 8 核    | 16 GB    | 无特殊要求       | 千兆网卡 | 10.168.2.153 |
| Master2  | 8 核    | 16 GB    | 无特殊要求       | 千兆网卡 | 10.168.3.60  |
| Master3  | 8 核    | 16 GB    | 无特殊要求       | 千兆网卡 | 10.168.2.209 |
| Node1    | 48 核   | 128 GB   | 1T HDD, 800G SSD | 千兆网卡 | 10.168.3.53  |
| Node2    | 48 核   | 128 GB   | 1T HDD, 800G SSD | 千兆网卡 | 10.168.3.55  |
| Node3    | 48 核   | 128GB    | 1T HDD, 800G SSD | 千兆网卡 | 10.168.3.30  |

### 软件版本

| 软件名称   | 版本                                    |
| :--------- | :-------------------------------------- |
| Docker     | Docker CE 19.03.9                       |
| Kubernetes | v1.18.6                                 |
| CentOS     | CentOS 7.9，内核 3.10.0-1160.el7.x86_64 |
| Helm       | v3.2.4                                  |

如下配置在部署K8S集群时基本已设置完毕，如有未修改的请根据以下修改；

建议关闭防火墙：

```shell
systemctl stop firewalld
systemctl disable firewalld
```

如果无法关闭 firewalld 服务，为了保证 Kubernetes 正常运行，需要打开以下端口：

1. 在 Master 节点上，打开以下端口，然后重新启动服务：

   ```shell
   firewall-cmd --permanent --add-port=6443/tcp
   firewall-cmd --permanent --add-port=2379-2380/tcp
   firewall-cmd --permanent --add-port=10250/tcp
   firewall-cmd --permanent --add-port=10251/tcp
   firewall-cmd --permanent --add-port=10252/tcp
   firewall-cmd --permanent --add-port=10255/tcp
   firewall-cmd --permanent --add-port=8472/udp
   firewall-cmd --add-masquerade --permanent
   
   # 当需要在 Master 节点上暴露 NodePort 时候设置
   firewall-cmd --permanent --add-port=30000-32767/tcp
   
   systemctl restart firewalld
   ```

2. 在 Node 节点上，打开以下端口，然后重新启动服务：

   ```shell
   firewall-cmd --permanent --add-port=10250/tcp
   firewall-cmd --permanent --add-port=10255/tcp
   firewall-cmd --permanent --add-port=8472/udp
   firewall-cmd --permanent --add-port=30000-32767/tcp
   firewall-cmd --add-masquerade --permanent
   
   systemctl restart firewalld
   ```

### 配置 Iptables

FORWARD 链默认配置成 ACCEPT，并将其设置到开机启动脚本里：

```shell
iptables -P FORWARD ACCEPT
```

### 禁用 SELinux

```shell
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### 关闭 Swap

Kubelet 正常工作需要关闭 Swap，并且把 `/etc/fstab` 里面有关 Swap 的那行注释掉：

```shell
swapoff -a
sed -i 's/^\(.*swap.*\)$/#\1/' /etc/fstab 
```

### 内核参数设置

按照下面的配置设置内核参数，也可根据自身环境进行微调：

```shell
modprobe br_netfilter

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
net.core.somaxconn = 32768
vm.swappiness = 0
net.ipv4.tcp_syncookies = 0
net.ipv4.ip_forward = 1
fs.file-max = 1000000
fs.inotify.max_user_watches = 1048576
fs.inotify.max_user_instances = 1024
net.ipv4.conf.all.rp_filter = 1
net.ipv4.neigh.default.gc_thresh1 = 80000
net.ipv4.neigh.default.gc_thresh2 = 90000
net.ipv4.neigh.default.gc_thresh3 = 100000
EOF

sysctl --system
```

### 配置 Irqbalance 服务

[Irqbalance](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/performance_tuning_guide/sect-red_hat_enterprise_linux-performance_tuning_guide-tool_reference-irqbalance) 服务可以将各个设备对应的中断号分别绑定到不同的 CPU 上，以防止所有中断请求都落在同一个 CPU 上而引发性能瓶颈。

```shell
systemctl enable irqbalance
systemctl start irqbalance
```

### CPUfreq 调节器模式设置

为了让 CPU 发挥最大性能，请将 CPUfreq 调节器模式设置为 performance 模式。详细参考[在部署目标机器上配置 CPUfreq 调节器模式](https://pingcap.com/docs-cn/stable/online-deployment-using-ansible/#查看系统支持的调节器模式)。

```shell
cpupower frequency-set --governor performance
```

### Ulimit 设置

TiDB 集群默认会使用很多文件描述符，需要将工作节点上面的 `ulimit` 设置为大于等于 `1048576`：

```shell
cat <<EOF >>  /etc/security/limits.confroot        soft        nofile        1048576root        hard        nofile        1048576root        soft        stack         10240EOFsysctl --system
```

## 配置Storage Class

#### 本地 PV 配置

- 给监控数据使用的盘，可以参考[步骤](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner/blob/master/docs/operations.md#sharing-a-disk-filesystem-by-multiple-filesystem-pvs)挂载盘，创建目录，并将新建的目录以 bind mount 方式挂载到 `/mnt/disks` 目录，后续创建 `local-storage` `StorageClass`。

  > **注意：**
  >
  > 该步骤中创建的目录个数取决于规划的 TiDB 集群数量。1 个目录会对应创建 1 个 PV。每个 TiDB 集群的监控数据会使用 1 个 PV。

- 给 TiDB Binlog 和备份数据使用的盘，可以和监控数据共同使用。

- 给 PD 数据使用的盘，可以参考[步骤](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner/blob/master/docs/operations.md#sharing-a-disk-filesystem-by-multiple-filesystem-pvs)挂载盘，创建目录，并将新建的目录以 bind mount 方式挂载到 `/mnt/sharedssd` 目录，后续创建 `shared-ssd-storage` `StorageClass`。

  > **注意：**
  >
  > 该步骤中创建的目录个数取决于规划的 TiDB 集群数量及每个集群内的 PD 数量。1 个目录会对应创建 1 个 PV。每个 PD 会使用一个 PV。

- 给 TiKV 数据使用的盘，可通过[普通挂载](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner/blob/master/docs/operations.md#use-a-whole-disk-as-a-filesystem-pv)方式将盘挂载到 `/mnt/ssd` 目录，后续创建 `ssd-storage` `StorageClass`。

盘挂载完成后，需要根据上述磁盘挂载情况修改 [`local-volume-provisioner` yaml 文件](https://raw.githubusercontent.com/pingcap/tidb-operator/master/manifests/local-dind/local-volume-provisioner.yaml)，配置发现目录并创建必要的 `StorageClass`。以下是根据上述挂载修改的 yaml 文件示例：

```yaml
apiVersion: storage.k8s.io/v1kind: StorageClassmetadata:  name: "local-storage"provisioner: "kubernetes.io/no-provisioner"volumeBindingMode: "WaitForFirstConsumer"---apiVersion: storage.k8s.io/v1kind: StorageClassmetadata:  name: "ssd-storage"provisioner: "kubernetes.io/no-provisioner"volumeBindingMode: "WaitForFirstConsumer"---apiVersion: storage.k8s.io/v1kind: StorageClassmetadata:  name: "shared-ssd-storage"provisioner: "kubernetes.io/no-provisioner"volumeBindingMode: "WaitForFirstConsumer"---apiVersion: v1kind: ConfigMapmetadata:  name: local-provisioner-config  namespace: kube-systemdata:  nodeLabelsForPV: |    - kubernetes.io/hostname  storageClassMap: |    shared-ssd-storage:      hostDir: /mnt/sharedssd      mountDir: /mnt/sharedssd    ssd-storage:      hostDir: /mnt/ssd      mountDir: /mnt/ssd    local-storage:      hostDir: /mnt/disks      mountDir: /mnt/disks---......          volumeMounts:            ......            - mountPath: /mnt/ssd              name: local-ssd              mountPropagation: "HostToContainer"            - mountPath: /mnt/sharedssd              name: local-sharedssd              mountPropagation: "HostToContainer"            - mountPath: /mnt/disks              name: local-disks              mountPropagation: "HostToContainer"      volumes:        ......        - name: local-ssd          hostPath:            path: /mnt/ssd        - name: local-sharedssd          hostPath:            path: /mnt/sharedssd        - name: local-disks          hostPath:            path: /mnt/disks......
```

##### 根据配置文件创建挂载点

#开始尽量创建多个挂载点，每个挂载点生成一个pv

格式化磁盘：

```shell
mkfs.ext4 /dev/sdbecho /dev/sdb /tidb/disks ext4 defaults 0 0 | sudo tee -a /etc/fstabmount -a df -h
```

将挂载点挂载到发现目录：

```
for i in $(seq 1 30); do  sudo mkdir -p /tidb/disks/local${i} /mnt/disks/local${i}  sudo mount --bind /tidb/disks/local${i} /mnt/disks/local${i}done#写入挂载目录，开机自动挂载for i in $(seq 1 30); do  echo /tidb/disks/local${i} /mnt/disks/local${i} none bind 0 0 | sudo tee -a /etc/fstabdonemount adf -h
```

##ssd和sharedssd也是同样操作，修改磁盘目录和挂载目录名称即可

最后通过 `kubectl apply` 命令部署 `local-volume-provisioner` 程序。

```
kubectl apply -f local-volume-provisioner.yaml
```

后续创建 TiDB 集群或备份等组件的时候，再配置相应的 `StorageClass` 供其使用。



## 部署 TiDB Operator

### 创建 CRD

TiDB Operator 使用 [Custom Resource Definition (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions) 扩展 Kubernetes，所以要使用 TiDB Operator，必须先创建 `TidbCluster` 自定义资源类型。只需要在你的 Kubernetes 集群上创建一次即可：

```shell
kubectl apply -f https://raw.githubusercontent.com/pingcap/tidb-operator/v1.1.12/manifests/crd.yaml
```

如果服务器没有外网，需要先用有外网的机器下载 `crd.yaml` 文件，然后再进行安装：

```shell
wget https://raw.githubusercontent.com/pingcap/tidb-operator/v1.1.12/manifests/crd.yamlkubectl apply -f ./crd.yaml
```

如果显示如下信息表示 CRD 安装成功：

```shell
kubectl get crdNAME                                 CREATED ATbackups.pingcap.com                  2020-06-11T07:59:40Zbackupschedules.pingcap.com          2020-06-11T07:59:41Zrestores.pingcap.com                 2020-06-11T07:59:40Ztidbclusterautoscalers.pingcap.com   2020-06-11T07:59:42Ztidbclusters.pingcap.com             2020-06-11T07:59:38Ztidbinitializers.pingcap.com         2020-06-11T07:59:42Ztidbmonitors.pingcap.com             2020-06-11T07:59:41Z
```

### 自定义部署 TiDB Operator

若需要快速部署 TiDB Operator，可参考快速上手中[部署 TiDB Operator文档](https://docs.pingcap.com/zh/tidb-in-kubernetes/stable/get-started#部署-tidb-operator)。本节介绍自定义部署 TiDB Operator 的配置方式。

创建 CRDs 之后，在 Kubernetes 集群上部署 TiDB Operator有两种方式：在线和离线部署。

#### 在线部署 TiDB Operator

1. 获取你要部署的 `tidb-operator` chart 中的 `values.yaml` 文件：

   ```shell
   mkdir -p ${HOME}/tidb-operator && \helm inspect values pingcap/tidb-operator --version=${chart_version} > ${HOME}/tidb-operator/values-tidb-operator.yaml
   ```

   > **注意：**
   >
   > `${chart_version}` 在后续文档中代表 chart 版本，例如 `v1.1.12`，可以通过 `helm search repo -l tidb-operator` 查看当前支持的版本。

2. 配置 TiDB Operator

   TiDB Operator 里面会用到 `k8s.gcr.io/kube-scheduler` 镜像，如果无法下载该镜像，可以修改 `${HOME}/tidb-operator/values-tidb-operator.yaml` 文件中的 `scheduler.kubeSchedulerImageName` 为 `registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler`。

   其他项目例如：`limits`、`requests` 和 `replicas`，请根据需要进行修改。

3. 部署 TiDB Operator

   ```shell
   helm install tidb-operator pingcap/tidb-operator --namespace=tidb-admin --version=${chart_version} -f ${HOME}/tidb-operator/values-tidb-operator.yaml && \kubectl get po -n tidb-admin -l app.kubernetes.io/name=tidb-operator
   ```

   > **注意：**
   >
   > 如果对应 `tidb-admin` namespace 不存在，则可先使用 `kubectl create namespace tidb-admin` 创建该 namespace。

4. 升级 TiDB Operator

   如果需要升级 TiDB Operator，请先修改 `${HOME}/tidb-operator/values-tidb-operator.yaml` 文件，然后执行下面的命令进行升级：

   ```shell
   helm upgrade tidb-operator pingcap/tidb-operator -f  ${HOME}/tidb-operator/values-tidb-operator.yaml
   ```

#### 离线安装 TiDB Operator

如果服务器没有外网，需要按照下面的步骤来离线安装 TiDB Operator：

1. 下载 `tidb-operator` chart

   如果服务器上没有外网，就无法通过配置 Helm repo 来安装 TiDB Operator 组件以及其他应用。这时，需要在有外网的机器上下载集群安装需用到的 chart 文件，再拷贝到服务器上。

   通过以下命令，下载 `tidb-operator` chart 文件：

   ```shell
   wget http://charts.pingcap.org/tidb-operator-v1.1.12.tgz
   ```

   将 `tidb-operator-v1.1.12.tgz` 文件拷贝到服务器上并解压到当前目录：

   ```shell
   tar zxvf tidb-operator.v1.1.12.tgz
   ```

2. 下载 TiDB Operator 运行所需的 Docker 镜像

   如果服务器没有外网，需要在有外网的机器上将 TiDB Operator 用到的所有 Docker 镜像下载下来并上传到服务器上，然后使用 `docker load` 将 Docker 镜像安装到服务器上。

   TiDB Operator 用到的 Docker 镜像有：

   ```shell
   pingcap/tidb-operator:v1.1.12pingcap/tidb-backup-manager:v1.1.12bitnami/kubectl:latestpingcap/advanced-statefulset:v0.3.3k8s.gcr.io/kube-scheduler:v1.16.9
   ```

   其中 `k8s.gcr.io/kube-scheduler:v1.16.9` 请跟你的 Kubernetes 集群的版本保持一致即可，不用单独下载。

   接下来通过下面的命令将所有这些镜像下载下来：

   ```shell
   docker pull pingcap/tidb-operator:v1.1.12docker pull pingcap/tidb-backup-manager:v1.1.12docker pull bitnami/kubectl:latestdocker pull pingcap/advanced-statefulset:v0.3.3docker save -o tidb-operator-v1.1.12.tar pingcap/tidb-operator:v1.1.12docker save -o tidb-backup-manager-v1.1.12.tar pingcap/tidb-backup-manager:v1.1.12docker save -o bitnami-kubectl.tar bitnami/kubectl:latestdocker save -o advanced-statefulset-v0.3.3.tar pingcap/advanced-statefulset:v0.3.3
   ```

   接下来将这些 Docker 镜像上传到服务器上，并执行 `docker load` 将这些 Docker 镜像安装到服务器上：

   ```shell
   docker load -i tidb-operator-v1.1.12.tardocker load -i tidb-backup-manager-v1.1.12.tardocker load -i bitnami-kubectl.tardocker load -i advanced-statefulset-v0.3.3.tar
   ```

3. 配置 TiDB Operator

   TiDB Operator 内嵌了一个 `kube-scheduler` 用来实现自定义调度器，请修改 `./tidb-operator/values.yaml` 文件来配置这个内置 `kube-scheduler` 组件的 Docker 镜像名字和版本，例如你的 Kubernetes 集群中的 `kube-scheduler` 使用的镜像为 `k8s.gcr.io/kube-scheduler:v1.16.9`，请这样设置 `./tidb-operator/values.yaml`：

   ```shell
   ...scheduler:  serviceAccount: tidb-scheduler  logLevel: 2  replicas: 1  schedulerName: tidb-scheduler  resources:    limits:      cpu: 250m      memory: 150Mi    requests:      cpu: 80m      memory: 50Mi  kubeSchedulerImageName: k8s.gcr.io/kube-scheduler  kubeSchedulerImageTag: v1.16.9...
   ```

   其他项目例如：`limits`、`requests` 和 `replicas`，请根据需要进行修改。

4. 安装 TiDB Operator

   使用下面的命令安装 TiDB Operator：

   ```shell
   helm install tidb-operator ./tidb-operator --namespace=tidb-admin
   ```

   > **注意：**
   >
   > 如果对应 `tidb-admin` namespace 不存在，则可先使用 `kubectl create namespace tidb-admin` 创建该 namespace。

5. 升级 TiDB Operator

   如果需要升级 TiDB Operator，请先修改 `./tidb-operator/values.yaml` 文件，然后执行下面的命令进行升级：

   ```shell
   helm upgrade tidb-operator ./tidb-operator
   ```

## 配置 TiDB 集群

先下载示例的tidb-cluster.yaml文件，然后修改配置文件中的各组件的存储类型，相应配置；

#我们所使用的是多盘挂载

#### 多盘挂载

TiDB Operator 支持为 PD、TiDB、TiKV 挂载多块 PV，可以用于不同用途的数据写入。

每个组件都可以配置 `storageVolumes` 字段，用于描述用户自定义的多个 PV。

> **注意：**
>
> 你需要在集群创建之前配置 `storageVolumes`。集群创建完成后，不支持添加或者删除 `storageVolumes`。对于已经配置的 `storageVolumes`，除增大 `storageVolume.storageSize` 外，其他项不支持修改。如果要增大 `storageVolume.storageSize`，需要对应的 StorageClass 支持[动态扩容](https://kubernetes.io/blog/2018/07/12/resizing-persistent-volumes-using-kubernetes/)。

相关字段的含义如下：

- `storageVolume.name`：PV 的名称。

- `storageVolume.storageClassName`：PV 使用哪一个 StorageClass。如果不配置，会使用 spec.pd/tidb/tikv.storageClassName。

- `storageVolume.storageSize`：申请 PV 存储容量的大小。

- `storageVolume.mountPath`：将 PV 挂载到容器的哪个目录。

  

  ```
  pd:    baseImage: pingcap/pd    replicas:     # if storageClassName is not set, the default Storage Class of the Kubernetes cluster will be used    # storageClassName: local-storage    requests:      storage: "500Gi"    config:      log:        file:          filename: /var/log/pdlog/pd.log        level: "warn"    storageVolumes:      - name: log        storageSize: "20Gi"        mountPath: "/var/log/pdlog"  tidb:    baseImage: pingcap/tidb    replicas: 3    service:      type: ClusterIP    config:      log:        file:          filename: /var/log/tidblog/tidb.log        level: "warn"    storageVolumes:      - name: log        storageSize: "20Gi"        mountPath: "/var/log/tidblog"  tikv:    baseImage: pingcap/tikv    replicas: 3    # if storageClassName is not set, the default Storage Class of the Kubernetes cluster will be used    # storageClassName: local-storage    requests:      storage: "400Gi"    config:      storage:        # In basic examples, you can set this to avoid using too much storage.        reserve-space: "0MB"      rocksdb:        wal-dir: "/data_sbi/tikv/wal"      titan:        dirname: "/data_sbj/titan/data"    storageVolumes:      - name: wal        storageSize: "20Gi"        mountPath: "/data_sbi/tikv/wal"      - name: titan        storageSize: "20Gi"        mountPath: "/data_sbj/titan/data"
  ```

**注意：**

TiDB Operator 默认会使用一些挂载路径，比如会为 TiDB Pod 挂载 `EmptyDir` 到 `/var/log/tidb` 目录。在配置 `storageVolumes` 的时候要避免配置重复的 `mountPath`。

默认示例的集群拓扑是：3 个 PD Pod，3 个 TiKV Pod，2 个 TiDB Pod。在该部署拓扑下根据数据高可用原则，TiDB Operator 扩展调度器要求 Kubernetes 集群中至少有 3 个节点。可以修改 `replicas` 配置来更改每个组件的 Pod 数量。

## 部署 TiDB 集群

1. 创建 `Namespace`：

   ```shell
   kubectl create namespace aimo-tidb
   ```

   > **注意：**
   >
   > `namespace` 是[命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)，可以起一个方便记忆的名字，比如和 `cluster_name` 相同的名称。

2. 部署 TiDB 集群：

   ```shell
   kubectl apply -f tidb-cluster.yaml -n aimo-tidb
   ```

   > 建议在 `/opt` 目录下组织 TiDB 集群的配置，并将其另存为 `/opt/tidb-cluster.yaml`。默认条件下，修改配置不会自动应用到 TiDB 集群中，只有在 Pod 重启时，才会重新加载新的配置文件。

3. 通过下面命令查看 Pod 状态：

   `kubectl get po -n aimo-tidb -l app.kubernetes.io/instance=advanced-tidb`





## 访问 TiDB 集群

Service 可以根据场景配置不同的类型，比如 `ClusterIP`、`NodePort`、`LoadBalancer` 等，对于不同的类型可以有不同的访问方式。

可以通过如下命令获取 TiDB Service 信息：

````shell
[root@db-master-1 ~]# kubectl get svc -n aimo-tidbNAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGEadvanced-tidb-discovery          ClusterIP   172.16.255.22    <none>        10261/TCP,10262/TCP              7d2hadvanced-tidb-grafana            NodePort    172.16.255.16    <none>        3000:31575/TCP                   7dadvanced-tidb-monitor-reloader   NodePort    172.16.255.156   <none>        9089:32193/TCP                   7dadvanced-tidb-pd                 NodePort    172.16.255.55    <none>        2379:32379/TCP                   7d2hadvanced-tidb-pd-peer            ClusterIP   None             <none>        2380/TCP                         7d2hadvanced-tidb-prometheus         NodePort    172.16.255.189   <none>        9090:30698/TCP                   7dadvanced-tidb-tidb               NodePort    172.16.255.138   <none>        4000:32429/TCP,10080:32421/TCP   7d2hadvanced-tidb-tidb-peer          ClusterIP   None             <none>        10080/TCP                        7d2hadvanced-tidb-tikv-peer          ClusterIP   None             <none>        20160/TCP                        7d2h
````

3000:31575/TCP #3000就是pod里面服务开放的端口，31575就是经过pod所在节点开放出来的端口，外部可通过此端口访问到pod

#### ClusterIP

`ClusterIP` 是通过集群的内部 IP 暴露服务，选择该类型的服务时，只能在集群内部访问，可以通过如下方式访问：

- ClusterIP + ServicePort
- Service 域名 (`${serviceName}.${namespace}`) + ServicePort

#### NodePort

NodePort 是通过节点的 IP 和静态端口暴露服务。通过请求 `NodeIP + NodePort`，可以从集群的外部访问一个 NodePort 服务。

## 使用 Ingress 访TIDB

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) 是一个 API 对象，负责管理集群中服务的外部访问。

ingress的工作原理
ingress具体的工作原理如下:
step1：ingress contronler通过与k8s的api进行交互，动态的去感知k8s集群中ingress服务规则的变化，然后读取它，并按照定义的ingress规则，转发到k8s集群中对应的service。
step2：而这个ingress规则写明了哪个域名对应k8s集群中的哪个service，然后再根据ingress-controller中的nginx配置模板，生成一段对应的nginx配置。
step3：然后再把该配置动态的写到ingress-controller的pod里，该ingress-controller的pod里面运行着一个nginx服务，控制器会把生成的nginx配置写入到nginx的配置文件中，然后reload一下，使其配置生效,以此来达到域名分配置及动态更新的效果。

### 环境准备

使用 `Ingress` 前，需要在 Kubernetes 集群中安装 `Ingress` 控制器，否则仅创建 `Ingress` 资源无效。你可能需要部署 `Ingress` 控制器，例如 [ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/)。你可以从许多 [Ingress 控制器](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)中进行选择。

#在部署集群时已经部署了nginx-ingress

创建一个tidb-ingress.yaml文件

```yaml
apiVersion: extensions/v1beta1kind: Ingressmetadata:  name: advanced-tidb  namespace: aimo-tidbspec:  rules:    - host: aimo.tidb.cn      http:        paths:          - backend:              serviceName: advanced-tidb-discovery              servicePort: 10262            path: /dashboard
```

```shell
kubectl apply -f tidb-ingress.yaml -n aimo-tidb# 查看创建结果[root@db-master-1 tidb]# kubectl get ingress -n aimo-tidbNAME            CLASS    HOSTS                                  ADDRESS          PORTS   AGEadvanced-tidb   <none>   aimo.tidb.cn                           172.16.255.149   80      7d2h# 通过域名访问需要修改本地hosts或做内部DNS解析
```





## TiDB 集群的监控

TiDB 通过 Prometheus 和 Grafana 监控 TiDB 集群。在通过 TiDB Operator 创建新的 TiDB 集群时，可以对于每个 TiDB 集群，创建、配置一套独立的监控系统，与 TiDB 集群运行在同一 Namespace，包括 Prometheus 和 Grafana 两个组件。

下载官方的tidb-operator.yaml文件进行修改配置；

### 持久化监控数据

可以在 `TidbMonitor` 中设置 `spec.persistent` 为 `true` 来持久化监控数据。开启此选项时应将 `spec.storageClassName` 设置为一个当前集群中已有的存储，并且此存储应当支持将数据持久化，否则会存在数据丢失的风险。配置示例如下：

``` persistent: true
  persistent: true  storageClassName: local-storage  storage: 10Gi
```

```shell
kubectl apply -f tidb-monitor.yaml -n aimo-tidb# 查看pvc[root@db-master-1 tidb]# kubectl get pvc -n aimo-tidb | grep monitoradvanced-tidb-monitor             Bound    local-pv-523e04     1007Gi     RWO            local-storage        7d# 查看pod启动情况[root@db-master-1 tidb]# kubectl get pods -n aimo-tidb | grep monitoradvanced-tidb-monitor-654888cb88-zxmch     3/3     Running   0          7d1h
```

### 使用 Ingress 访问 TidbMonitor

创建monitor-ingress.ymal文件；

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: monitor-tidb
  namespace: aimo-tidb
spec:
  rules:
    - host: tidb.grafana.com
      http:
        paths:
          - path: /
            backend:
              serviceName: advanced-tidb-grafana
              servicePort: 3000
    - host: tidb.prometheus.com
      http:
        paths:
          - path: /
            backend:
              serviceName: advanced-tidb-prometheus
              servicePort: 9090
```

```shell
# 应用yaml文件
kubectl apply -f monitor-ingress.yaml -n aimo-tidb# 查看ingress[root@db-master-1 tidb]# kubectl get ingress -n aimo-tidbNAME            CLASS    HOSTS                                  ADDRESS          PORTS   AGEadvanced-tidb   <none>   aimo.tidb.cn                           172.16.255.149   80      7d2hmonitor-tidb    <none>   tidb.grafana.com,tidb.prometheus.com   172.16.255.149   80      6d23h
```


