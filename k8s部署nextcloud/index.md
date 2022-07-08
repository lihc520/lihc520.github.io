# K8s部署Nextcloud


### 部署 NFS 服务

#### 在NFS服务器上安装 NFS 与 rpcbind 服务

```bash
#   创建 NFS 存储目录
mkdir -p /data/NFS
#   安装nfs服务
yum -y install nfs-utils rpcbind
#   修改配置文件
echo "/data/NFS *(rw,sync,no_root_squash,no_subtree_check)" > /etc/exports
#   启动服务
systemctl start nfs && systemctl start rpcbind
#   设置开机启动
systemctl enable nfs-server && systemctl enable rpcbindCOPY
```

#### 集群节点安装 nfs

重点全部节点都需要安装 nfs-utils 安装了即可不需要配置，否则节点无法挂载 pv

```
#   安装nfs服务
yum -y install nfs-utils
```

### 在master上操作创建静态pv、pvc

#### 创建nextcloud命名空间

```
kubectl create ns nextcloud
```

#### 创建pv

```
## 创建mysql的pv文件
cat > nfs-mysql.yaml << EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-mysql
  labels:
    pv: nfs-mysql
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs-nextcloud
  nfs:
    path: /data/NFS/mysql
    server: 10.168.3.158
EOF
## 应用文件
kubectl apply -f nfs-mysql.yaml -n nextcloud

## 创建nextcloud的pv文件
cat > nfs-nextcloud.yaml << EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-nextcloud
  labels:
    pv: nfs-nextcloud
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /data/NFS/nextcloud
    server: 10.168.3.158
EOF
## 应用文件
kubectl apply -f nfs-nextcloud.yaml -n nextcloud

```

#### 创建pvc

```
## 创建mysql的pvc文件
cat > nfs-mysql-pvc.yaml << EOF	
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-mysql
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-nextcloud
  selector:
    matchLabels:
      pv: nfs-mysqld
EOF
## 应用文件
kubectl apply -f nfs-mysql-pvc.yaml -n nextcloud


## 创建nextcloud的pvc文件
cat > nfs-cloud-pvc.yaml << EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-cloud
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: nfs-nextcloud
  selector:
    matchLabels:
      pv: nfs-nextcloud
EOF	
## 应用文件
kubectl apply -f nfs-cloud-pvc.yaml -n nextcloud
```

#### 部署nextcloud的web端

```
## 创建yaml文件
cat > nextcloud-server.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud-server
  labels:
    app: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      pod-label: nextcloud-server-pod
  template:
    metadata:
      labels:
        pod-label: nextcloud-server-pod
    spec:
      containers:
      - name: nextcloud
        image: nextcloud:21.0.0-apache
        volumeMounts:
        - name: nfs-cloud
          mountPath: /var/www/html
          subPath: server-data
      volumes:
      - name: nfs-cloud
        persistentVolumeClaim:
          claimName: nfs-cloud
---
apiVersion: v1
kind: Service
metadata:
  name: nextcloud-server
  labels:
    app: nextcloud
spec:
  selector:
    pod-label: nextcloud-server-pod
  ports:
  - protocol: TCP
    port: 80
EOF

## 应用文件
kubectl apply -f nextcloud-server.yaml -n nextcloud
```

#### 创建mysql用户密码在nextcloud空间

```bash
kubectl create secret generic nextcloud-db-secret \
    --from-literal=MYSQL_ROOT_PASSWORD=password \
    --from-literal=MYSQL_USER=nextcloud \
    --from-literal=MYSQL_PASSWORD=password -n nextcloud
```

#### 部署MySQL

```bash
## 创建mysql的yaml文件
cat > nextcloud-db.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud-db
  labels:
    app: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      pod-label: nextcloud-db-pod
  template:
    metadata:
      labels:
        pod-label: nextcloud-db-pod
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_DATABASE
          value: nextcloud
        envFrom:
        - secretRef:
            name: nextcloud-db-secret
        volumeMounts:
        - name: nfs-mysql
          mountPath: /var/lib/mysql
          subPath: mysql-data
      volumes:
      - name: nfs-mysql
        persistentVolumeClaim:
          claimName: nfs-mysql
---
apiVersion: v1
kind: Service
metadata:
  name: nextcloud-db
  labels:
    app: nextcloud
spec:
  selector:
    pod-label: nextcloud-db-pod
  ports:
  - protocol: TCP
    port: 3306
EOF
## 应用文件
kubectl apply -f nextcloud-db.yaml -n nextcloud
```

#### 查看部署情况

```bash
[root@master3 nextcloud]# kubectl get pvc -n nextcloud
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
nfs-cloud   Bound    pvc-0d2e93d7-ed93-487e-88b1-10f7a0fe37a8   90Gi       RWX            nfs-nextcloud   3d19h
nfs-mysql   Bound    nfs-mysql                                  10Gi       RWX            nfs-nextcloud   3d2h

[root@master3 nextcloud]# kubectl get pod -n nextcloud
NAME                                READY   STATUS    RESTARTS   AGE
nextcloud-db-5cb8b84dc9-ctnts       1/1     Running   0          2d2h
nextcloud-server-648bb47b66-rmch5   1/1     Running   0          2d2h

[root@master3 nextcloud]# kubectl get svc -n nextcloud
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nextcloud-db       ClusterIP   10.111.175.129   <none>        3306/TCP       3d2h
nextcloud-server   NodePort    10.99.17.67      <none>        80:31800/TCP   3d19h
```

都正常后到浏览器输入随便一个node节点的ip:31800进入nextcloud页面；

数据库帐号密码填刚刚创建的，数据库地址填nextcloud-db的ClusterIP加3306端口即可。

