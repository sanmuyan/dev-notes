# Rock ceph 生产部署

## 节点规划

- OS `Ubuntu 22.04`

|K8s 节点|角色|标签|
|-|-|-|
|master-node-1|mon/mgr|ceph-mon-node=node/ceph-mgr-node=node/ceph-provisioner-node=node|
|master-node-2|mon/mgr|ceph-mon-node=node/ceph-mgr-node=node/ceph-provisioner-node=node|
|master-node-3|mon/mgr|ceph-mon-node=node/ceph-provisioner-node=node|
|storage-node-1|storage|ceph-storage-node=node|
|storage-node-2|storage|ceph-storage-node=node|
|storage-node-3|storage|ceph-storage-node=node|

```shell
apt install nfs-common
```

## 安装部署

```shell
git clone --single-branch --branch v1.17.6 https://github.com/rook/rook.git


cp rook/deploy/charts/rook-ceph/values.yaml .
cp rook/deploy/examples/cluster.yaml .

helm repo add rook-release https://charts.rook.io/release
helm install \
--create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph -f values.yaml \
--version v1.17.6

kubectl apply -f cluster.yaml
```

`values.yaml` Helm 安装配置，这些配置安装后会体现在 `cm/rook-ceph-operator-config`

```yaml
tolerations:
  - key: ceph-node
    operator: Exists
    effect: NoSchedule
csi:
  provisionerTolerations:
    - key: ceph-node
      operator: Exists
      effect: NoSchedule
  provisionerNodeAffinity: ceph-provisioner-node=node
  pluginTolerations:
    - key: ceph-node
      operator: Exists
      effect: NoSchedule
  pluginNodeAffinity: #
  nfs:
    enabled: true

discover:
  tolerations:
    - key: ceph-node
      operator: Exists
      effect: NoSchedule
  nodeAffinity: ceph-storage-node=node
```

`cluster.yaml` 配置 ceph 组件调度配置和存储节点硬盘绑定

```yaml
spec:
  placement:
    mgr:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: ceph-mgr-node
                  operator: In
                  values:
                    - node
      podAffinity:
      podAntiAffinity:
      topologySpreadConstraints:
      tolerations:
        - key: ceph-node
          operator: Exists
    mon:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: ceph-mon-node
                  operator: In
                  values:
                    - node
      podAffinity:
      podAntiAffinity:
      topologySpreadConstraints:
      tolerations:
        - key: ceph-node
          operator: Exists
    all:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: ceph-storage-node
                  operator: In
                  values:
                    - node
      tolerations:
        - key: ceph-node
          operator: Exists
  storage:
    # 关闭自动配置
    useAllNodes: false
    useAllDevices: false
    nodes:
      - name: "storage-node-1"
        devices:
          - name: "sdc"
      - name: "storage-node-2"
        devices:
          - name: "sdc"
      - name: "storage-node-3"
        deviceFilter: "^sd.*$"
```

## 安装验证

查看所有 Pod

```shell
kubectl -n rook-ceph get pod                                                                                                   
NAME                                               READY   STATUS      RESTARTS      AGE
csi-cephfsplugin-5mlfd                             3/3     Running     0             21h
csi-cephfsplugin-gxvg4                             3/3     Running     0             21h
csi-cephfsplugin-kl24t                             3/3     Running     0             21h
csi-cephfsplugin-provisioner-fd9f48fdd-4p5n8       6/6     Running     0             21h
csi-cephfsplugin-provisioner-fd9f48fdd-ffw2n       6/6     Running     0             91m
csi-nfsplugin-4jmzg                                3/3     Running     0             21h
csi-nfsplugin-cd4j4                                3/3     Running     0             21h
csi-nfsplugin-hcm6b                                3/3     Running     0             21h
csi-nfsplugin-provisioner-6f79b8d8cc-rp7x6         6/6     Running     0             91m
csi-nfsplugin-provisioner-6f79b8d8cc-vp4kn         6/6     Running     0             99m
csi-rbdplugin-6nxmp                                3/3     Running     0             21h
csi-rbdplugin-provisioner-69f8bb5544-9mxtj         6/6     Running     0             91m
csi-rbdplugin-provisioner-69f8bb5544-gtsjz         6/6     Running     0             21h
csi-rbdplugin-qdr9z                                3/3     Running     0             21h
csi-rbdplugin-sz5xv                                3/3     Running     0             21h
rook-ceph-crashcollector-node-2-7b566b7945-xnvfv   1/1     Running     0             42m
rook-ceph-crashcollector-node-3-8598f964-25npm     1/1     Running     0             42m
rook-ceph-crashcollector-node-4-57d4cfcb89-xk2z6   1/1     Running     0             40m
rook-ceph-exporter-node-2-67df888479-nksxw         1/1     Running     0             41m
rook-ceph-exporter-node-3-79c4d77cc8-4hs8w         1/1     Running     0             42m
rook-ceph-exporter-node-4-784bb7957f-w9q78         1/1     Running     0             40m
rook-ceph-mds-test-a-b9d7545ff-th7zn               2/2     Running     0             40m
rook-ceph-mds-test-b-69564c9f8c-8t8gl              2/2     Running     0             40m
rook-ceph-mgr-a-594bd5ccdb-k54wn                   3/3     Running     0             20h
rook-ceph-mgr-b-6b986c9b59-cts79                   3/3     Running     0             20h
rook-ceph-mon-a-6b7f7bfbd9-x6xqk                   2/2     Running     0             21h
rook-ceph-mon-b-c4b9cb6dc-hvw2z                    2/2     Running     0             21h
rook-ceph-mon-d-7659ff4959-d298s                   2/2     Running     0             91m
rook-ceph-operator-6cfd58b98-xvkdk                 1/1     Running     0             96m
rook-ceph-osd-0-6c854b675d-v9fxh                   2/2     Running     0             91m
rook-ceph-osd-1-d6b5c4478-j5gzz                    2/2     Running     0             91m
rook-ceph-osd-2-b49949c66-vdtj2                    2/2     Running     0             41m
rook-ceph-osd-3-59ddcc7f9b-49kgj                   2/2     Running     0             41m
rook-ceph-osd-4-d75784b8-mm2vp                     2/2     Running     0             41m
rook-ceph-osd-5-85c68b8d4b-vgk4z                   2/2     Running     0             41m
rook-ceph-osd-prepare-node-2-2d75v                 0/1     Completed   0             41m
rook-ceph-osd-prepare-node-3-z9pnh                 0/1     Completed   0             41m
rook-ceph-osd-prepare-node-4-gq856                 0/1     Completed   0             41m
```

Helm 安装后

- rook-ceph-operator 负责整个集群的自动化生命周期管理（部署、扩容、监控）

创建集群后

- rook-ceph-mon 集群状态，一致性、仲裁
- rook-ceph-mgr 集群监控与统计，管理入口
- rook-ceph-osd 存储数据管理，一个 osd 对用一个硬盘
- csi-cephfsplugin-provisioner 负责创建/删除 PVC 绑定的 CephFS 子卷
- csi-cephfsplugin 节点插件，运行在每个节点，负责将 CephFS 文件系统挂载到 Pod
- rook-ceph-exporter 通过 Prometheus 导出集群运行指标
- rook-ceph-crashcollector 收集故障日志，供后续分析或上传

查看集群状态

```shell
kubectl apply -f rook/deploy/examples/toolbox.yaml
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph status

  cluster:
    id:     3d0548ae-14eb-4ba0-818e-4254dfb5ffe9
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum a,b,d (age 116m)
    mgr: b(active, since 21h), standbys: a
    osd: 6 osds: 6 up (since 72m), 6 in (since 72m)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 2 objects, 449 KiB
    usage:   305 MiB used, 60 GiB / 60 GiB avail
    pgs:     1 active+clean

```

创建一个 SVC 访问 Dashboard

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: rook-ceph-mgr
    rook_cluster: rook-ceph
  name: rook-ceph-mgr-dashboard-external
  namespace: rook-ceph
spec:
  ports:
  - name: https-dashboard
    nodePort: 32170
    port: 8443
    protocol: TCP
    targetPort: 8443
  selector:
    app: rook-ceph-mgr
    mgr_role: active
    rook_cluster: rook-ceph
  sessionAffinity: None
  type: NodePort
```

```shell
# 获取密码
kubectl -n rook-ceph get secrets rook-ceph-dashboard-password -ojson |jq -r .data.password |base64 -d
```

访问 `https://192.168.1.1:32800`

## 配置 CFS

### 创建 CFS

`cfs-default.yaml`

```yaml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: cfs-default
  namespace: rook-ceph
spec:
  metadataPool:
    replicated:
      size: 3
  dataPools:
    - name: replicated
      replicated:
        size: 3
  preserveFilesystemOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
```

创建后会自动创建对应文件系统的 mds

```shell
kubectl apply -f cfs-default.yaml
kubectl apply -f cfs-default-sc.yaml

kubectl -n rook-ceph get pod |grep rook-ceph-mds-cfs-defaul
rook-ceph-mds-cfs-default-a-5654b85f5b-fjl4g       2/2     Running     0              5m16s
rook-ceph-mds-cfs-default-b-56cfd48488-5mvh6       2/2     Running     0              5m15s

kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph fs ls 

name: cfs-default, metadata pool: cfs-default-metadata, data pools: [cfs-default-replicated ]
```

### CSI 配置

`cfs-default-sc.yaml`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cfs-default
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"    
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  clusterID: rook-ceph
  fsName: cfs-default

  pool: cfs-default-replicated

  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
```

### 测试应用

`test-app.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cfs-test-pvc
  namespace: default
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cfs-default
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
  namespace: default
  labels:
    k8s-app: test-app
    kubernetes.io/cluster-service: "true"
spec:
  replicas: 2
  selector:
    matchLabels:
      k8s-app: test-app
  template:
    metadata:
      labels:
        k8s-app: test-app
    spec:
      containers:
      - name: app
        image: registry.cn-hangzhou.aliyuncs.com/sanmuyan/ubuntu:22.04-base-20250427
        imagePullPolicy: Always
        volumeMounts:
         - name: cfs-test
           mountPath: /cfs-test         
      volumes:
      - name: cfs-test
        persistentVolumeClaim:
          claimName: cfs-test-pvc
          readOnly: false
```

```shell
kubectl exec deployments/test-app -i -t -- df -Bg /cfs-test
Filesystem                                                                                                                                              1G-blocks  Used Available Use% Mounted on
10.43.181.233:6789,10.43.205.105:6789,10.43.224.151:6789:/volumes/csi/csi-vol-02d0b423-1332-49ff-b2ad-f6cdf33c5111/3f19f242-c5c9-4078-be10-94be06402a95        1G    0G        1G   0% /cfs-test

```

## 配置 NFS

### 导出 NFS

`nfs-server-default.yaml`

```yaml
apiVersion: ceph.rook.io/v1
kind: CephNFS
metadata:
  name: nfs-server-default
  namespace: rook-ceph
spec:
  server:
    active: 1
    placement:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: ceph-storage-node
              operator: In
              values:
              - node
    resources:
      limits:
        memory: "8Gi"
      requests:
       cpu: "2"
       memory: "2Gi"
      tolerations:
      - key: ceph-node
        operator: Exists
      podAffinity:
      podAntiAffinity:
    logLevel: NIV_INFO
---
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: nfs-server-default
  namespace: rook-ceph
spec:
  name: .nfs
  failureDomain: host
  replicated:
    size: 3
    requireSafeReplicaSize: true
```

```shell
kubectl apply -f nfs-server-default.yaml

# 10.43.4.13 就是 NFS 挂载地址
kubectl -n rook-ceph get svc rook-ceph-nfs-nfs-server-default-a
NAME                                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
rook-ceph-nfs-nfs-server-default-a   ClusterIP   10.43.4.13   <none>        2049/TCP,9587/TCP   4m6s
```

### 手动挂载

需要手动导出 NFS

```shell
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
# 查看刚才创建的 nfs cluster
ceph nfs cluster ls
# 在 cfs-default 上创建一个 subvolumegroup
ceph fs subvolumegroup create cfs-default nfs
# 在 cfs-default 上创建一个 subvolume
ceph fs subvolume create cfs-default nfs-server-default 0 nfs
# 查看 subvolume 的信息
ceph fs subvolume info cfs-default nfs-server-default --group_name nfs
# 在刚才创建的 subvolume 导出 nfs
ceph nfs export create cephfs nfs-server-default /nfsroot cfs-default /volumes/nfs/<subvolume_path>
```

导出完成后即可手动挂载 NFS `mount.nfs 10.43.4.13:/nfsroot /mnt`

### CSI 配置

`nfs-default-sc.yaml`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-default
provisioner: rook-ceph.nfs.csi.ceph.com
parameters:
  nfsCluster: nfs-server-default
  server: rook-ceph-nfs-nfs-server-default-a
  clusterID: rook-ceph
  fsName: cfs-default
  pool: cfs-default-replicated

  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
```

### 测试应用

`test-app.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-test-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-default
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
  namespace: default
  labels:
    k8s-app: test-app
    kubernetes.io/cluster-service: "true"
spec:
  replicas: 2
  selector:
    matchLabels:
      k8s-app: test-app
  template:
    metadata:
      labels:
        k8s-app: test-app
    spec:
      containers:
      - name: app
        image: registry.cn-hangzhou.aliyuncs.com/sanmuyan/ubuntu:22.04-base-20250427
        imagePullPolicy: Always
        volumeMounts:
         - name: nfs-test
           mountPath: /nfs-test
      volumes:
      - name: nfs-test
        persistentVolumeClaim:
          claimName: nfs-test-pvc
          readOnly: false
```

```yaml
kubectl exec deployments/test-app -i -t -- df -Bg /nfs-test
Filesystem                                                                                                    1G-blocks  Used Available Use% Mounted on
rook-ceph-nfs-nfs-server-default-a:/0001-0009-rook-ceph-0000000000000003-fb555e8c-68bf-4c4a-9b6a-6d7fe7bbc99f       19G    0G       19G   0% /nfs-test
```

## 配置 RBD

### 创建 RBD Pool

`rbdpool-default.yaml`

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: rbdpool-default
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
```

### CSI 配置

`rbd-default-sc.yaml`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rbd-default
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
    clusterID: rook-ceph
    pool: rbdpool-default
    imageFormat: "2"
    imageFeatures: layering
    csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
    csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
    csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
    csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
    csi.storage.k8s.io/fstype: ext4

reclaimPolicy: Delete
allowVolumeExpansion: true
```

### 测试应用

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: rbd-default
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
  namespace: default
  labels:
    k8s-app: test-app
    kubernetes.io/cluster-service: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: test-app
  template:
    metadata:
      labels:
        k8s-app: test-app
    spec:
      containers:
      - name: app
        image: registry.cn-hangzhou.aliyuncs.com/sanmuyan/ubuntu:22.04-base-20250427
        imagePullPolicy: Always
        volumeMounts:
         - name: rbd-test
           mountPath: /rbd-test
      volumes:
      - name: rbd-test
        persistentVolumeClaim:
          claimName: rbd-test-pvc
          readOnly: false
```

## 配置 COS

### 创建 COS

`cos-default.yaml`

```yaml
apiVersion: ceph.rook.io/v1
kind: CephObjectStore
metadata:
  name: cos-default
  namespace: rook-ceph
spec:
  metadataPool:
    failureDomain: host
    replicated:
      size: 3
  dataPool:
    failureDomain: host
    erasureCoded:
      dataChunks: 2
      codingChunks: 1
  preservePoolsOnDelete: true
  gateway:
    port: 80
    instances: 1
```

创建后会自动创建对应的 RGW

```shell
kubectl -n rook-ceph get pod |grep rgw
rook-ceph-rgw-cos-default-a-6fb45f9657-dzg6l              2/2     Running     0              4m40s
```

### 配置 CSI

`cos-bucket-default-sc.yaml`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: cos-bucket-default
provisioner: rook-ceph.ceph.rook.io/bucket
reclaimPolicy: Delete
parameters:
  objectStoreName: cos-default
  objectStoreNamespace: rook-ceph
```

### 创建 Bucket

`cos-bucket-test.yaml`

```yaml
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: cos-bucket-test
spec:
  generateBucketName: cos-bucket-test
  storageClassName: cos-bucket-default
```

### 访问测试

配置访问凭据

```shell
export AWS_HOST=$(kubectl -n default get cm cos-bucket-test -o jsonpath='{.data.BUCKET_HOST}')
export PORT=$(kubectl -n default get cm cos-bucket-test -o jsonpath='{.data.BUCKET_PORT}')
export BUCKET_NAME=$(kubectl -n default get cm cos-bucket-test -o jsonpath='{.data.BUCKET_NAME}')
export AWS_ACCESS_KEY_ID=$(kubectl -n default get secret cos-bucket-test -o jsonpath='{.data.AWS_ACCESS_KEY_ID}' | base64 --decode)
export AWS_SECRET_ACCESS_KEY=$(kubectl -n default get secret cos-bucket-test -o jsonpath='{.data.AWS_SECRET_ACCESS_KEY}' | base64 --decode)

mkdir ~/.aws
cat > ~/.aws/credentials << EOF
[default]
aws_access_key_id = ${AWS_ACCESS_KEY_ID}
aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}
EOF

```

获取访问端点

```shell
kubectl -n rook-ceph get svc rook-ceph-rgw-cos-default
NAME                        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
rook-ceph-rgw-cos-default   NodePort   10.43.36.117   <none>        80:32152/TCP   49m
```

访问测试

```shell
s5cmd --endpoint-url http://10.43.36.117 ls
2025/07/28 04:47:17  s3://cos-bucket-test-35bcb1ac-2a75-416f-a078-5aa44c8021fa

echo "test file" >test.txt

s5cmd --endpoint-url http://10.43.36.117 cp test.txt s3://cos-bucket-test-35bcb1ac-2a75-416f-a078-5aa44c8021fa
cp test.txt s3://cos-bucket-test-35bcb1ac-2a75-416f-a078-5aa44c8021fa/test.txt

s5cmd --endpoint-url http://10.43.36.117 cat s3://cos-bucket-test-35bcb1ac-2a75-416f-a078-5aa44c8021fa/test.txt
test file
```