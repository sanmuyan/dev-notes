# Cephadm 生产部署

## 节点规划

- OS `Ubuntu 22.04`

| K8s 节点 | IP            | 标签             |
| ------ | ------------- | -------------- |
| node-1 | 192.168.5.101 | _admin/mon/mgr |
| node-2 | 192.168.5.102 | _admin/mon/mgr |
| node-3 | 192.168.5.103 | _admin/mon     |
| node-4 | 192.168.5.104 | _storage       |
| node-5 | 192.168.5.105 | _storage       |
| node-6 | 192.168.5.106 | _storage       |

## 初始化集群

### 第一个节点

```shell
wget https://download.ceph.com/debian-squid/pool/main/c/ceph/cephadm_19.2.3-1jammy_amd64.deb
dpkg -i cephadm_19.2.3-1jammy_amd64.deb
cephadm add-repo --release squid
cephadm install ceph-common
cephadm bootstrap --mon-ip 192.168.5.101
...
Generating a dashboard self-signed certificate...
Creating initial admin user...
Fetching dashboard port number...
Ceph Dashboard is now available at:

             URL: https://node-1:8443/
            User: admin
        Password: 
...        
```

第一个节点初始化成功后，默认会安装

```shell
root@node-1:~# docker ps
CONTAINER ID   IMAGE                                     COMMAND                  CREATED             STATUS             PORTS     NAMES
2c8c7ccd91dd   quay.io/prometheus/alertmanager:v0.25.0   "/bin/alertmanager -…"   6 minutes ago       Up 6 minutes                 ceph-eef2c102-7692-11f0-b81f-00155d05320c-alertmanager-node-1
66d362062888   quay.io/prometheus/prometheus:v2.51.0     "/bin/prometheus --c…"   58 minutes ago      Up 58 minutes                ceph-eef2c102-7692-11f0-b81f-00155d05320c-prometheus-node-1
2f713a69ca40   quay.io/ceph/grafana:10.4.0               "/run.sh"                About an hour ago   Up About an hour             ceph-eef2c102-7692-11f0-b81f-00155d05320c-grafana-node-1
0fe33ee803f8   quay.io/prometheus/node-exporter:v1.7.0   "/bin/node_exporter …"   About an hour ago   Up About an hour             ceph-eef2c102-7692-11f0-b81f-00155d05320c-node-exporter-node-1
b317161f3207   quay.io/ceph/ceph                         "/usr/bin/ceph-crash…"   About an hour ago   Up About an hour             ceph-eef2c102-7692-11f0-b81f-00155d05320c-crash-node-1
792960f22907   quay.io/ceph/ceph                         "/usr/bin/ceph-expor…"   About an hour ago   Up About an hour             ceph-eef2c102-7692-11f0-b81f-00155d05320c-ceph-exporter-node-1
e15fbf3b70f9   quay.io/ceph/ceph:v19                     "/usr/bin/ceph-mgr -…"   About an hour ago   Up About an hour             ceph-eef2c102-7692-11f0-b81f-00155d05320c-mgr-node-1-lthlst
4ae5f24c237e   quay.io/ceph/ceph:v19                     "/usr/bin/ceph-mon -…"   About an hour ago   Up About an hour             ceph-eef2c102-7692-11f0-b81f-00155d05320c-mon-node-1
```
### 添加其他节点

```shell
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node-1
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node-2
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node-3
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node-4
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node-5

ceph orch host add node-2 192.168.5.102 --labels _admin
ceph orch host add node-3 192.168.5.103 --labels _admin

ceph orch host add node-4 192.168.5.104 --labels _storage
ceph orch host add node-5 192.168.5.105 --labels _storage
ceph orch host add node-6 192.168.5.106 --labels _storage
```
## MON 节点

```shell
ceph orch host label add node-1 mon
ceph orch host label add node-2 mon
ceph orch host label add node-3 mon
ceph orch apply mon --placement="3 label:mon"
```

## MGR 节点

```shell
ceph orch host label add node-1 mgr
ceph orch host label add node-2 mgr
ceph orch apply mgr --placement="2 label:mgr"
```

## 添加 OSD

```shell
ceph orch daemon add osd node-4:/dev/sdb
ceph orch daemon add osd node-5:/dev/sdb
ceph orch daemon add osd node-6:/dev/sdb
```

##  CFS 配置

### 创建

```shell
ceph orch host label add node-1 mds
ceph orch host label add node-2 mds
ceph orch host label add node-3 mds

ceph fs volume create cfs-default --placement="label:mds"
# 也可创建后
ceph orch apply mds cfs-default --placement="label:mds"
```

### 挂载

```shell
# 创建用户
ceph auth get-or-create client.cfsuser

# 客户端挂载
apt install ceph-common
mkdir -p -m 755 /etc/ceph
ssh root@node-1 "ceph config generate-minimal-conf" | tee /etc/ceph/ceph.conf
chmod 644 /etc/ceph/ceph.conf

ssh root@node-1 "ceph fs authorize cfs-default client.cfsuser / rw" | tee /etc/ceph/ceph.client.cfsuser.keyring
chmod 600 /etc/ceph/ceph.client.cfsuser.keyring
mkdir -p /mnt/cfs-test
mount -t ceph cfsuser@.cfs-default=/ /mnt/cfs-test

df -Bg /mnt/cfs-test
Filesystem                                                 Type 1G-blocks  Used Available Use% Mounted on
192.168.5.101:6789,192.168.5.102:6789,192.168.5.103:6789:/ ceph       95G    0G       95G   0% /mnt/cfs-test
```

## RBD 配置

### 创建

```shell
ceph osd pool create rbdpool-default
rbd pool init rbdpool-default
rbd create --size 1024 rbdpool-default/rbd-test
```

### 挂载

```shell
# 创建用户
ceph auth get-or-create client.rbduser mon 'profile rbd' osd 'profile rbd pool=rbdpool-default' mgr 'profile rbd pool=rbdpool-default'

# 客户端挂载
ssh root@node-1 "ceph auth get client.rbduser" | tee /etc/ceph/ceph.client.rbduser.keyring
chmod 600 /etc/ceph/ceph.client.rbduser.keyring

rbd device map rbdpool-default/rbd-test --id rbduser
mkfs.ext4  /dev/rbd0
mkdir -p /mnt/rbd-test
mount /dev/rbd0 /mnt/rbd-test/

df -Bg -T /mnt/rbd-test
Filesystem     Type 1G-blocks  Used Available Use% Mounted on
/dev/rbd0      ext4        1G    1G        1G   1% /mnt/rbd-test
```

## COS 配置

### 创建 RGW

```shell
ceph orch host label add node-3 rgw
ceph orch apply rgw rgw-default --placement="1 label:rgw"
```

### 访问 COS

```shell
radosgw-admin bucket create --bucket=cos-test --uid=cosuser
export AWS_ACCESS_KEY_ID=$(radosgw-admin user info --uid="cosuser" |jq -r '.keys[].access_key')
export AWS_SECRET_ACCESS_KEY=$(radosgw-admin user info --uid="cosuser" |jq -r '.keys[].secret_key')
mkdir ~/.aws
cat > ~/.aws/credentials << EOF
[default]
aws_access_key_id = ${AWS_ACCESS_KEY_ID}
aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}
EOF

cat > ~/.aws/config << EOF
[default]
region = default
output = json
EOF

s5cmd --endpoint-url  http://node-3 mb s3://cos-test
mb s3://cos-test

s5cmd --endpoint-url  http://node-3 ls
2025/08/11 14:08:07  s3://cos-test
```

## NFS 配置

### 创建  NFS Ganesha

```shell
ceph orch host label add node-3 nfs
ceph orch apply nfs nfs-server-default --placement="1 label:nfs"
```

### 导出 NFS

请参考 [[Rock Ceph 生产部署]]   导出 NFS
### 访问 NFS

```shell
mkdir -p /mnt/nfs-test
mount.nfs node-3:/nfsroot /mnt/nfs-test

df -Bg -T /mnt/nfs-test
Filesystem      Type 1G-blocks  Used Available Use% Mounted on
node-3:/nfsroot nfs4       87G    0G       87G   0% /mnt/nfs-test
```