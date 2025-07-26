# RKE2 高可用部署

## 节点规划

- OS `Ubuntu 22.04`

|节点|角色|IP|
|-|-|-|
|server-1|server|192.168.1.1|
|server-2|server|192.168.1.2|
|server-3|server|192.168.1.3|
|agent-1|agent|192.168.1.4|
|lb-1|lb|192.168.1.5|
|lb-2|lb|192.168.1.6|
|lb.server.loacl|vip|192.168.1.200|

## LB 配置

`lb-1 keepalived.conf`

```conf
global_defs {
   router_id lb-1
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }

    virtual_ipaddress {
        192.168.242.200
    }
    preempt delay 60
}
```

`lb-2 keepalived.conf`

```conf
global_defs {
   router_id lb-2
}
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }

    virtual_ipaddress {
        192.168.242.200
    }
    preempt delay 60
}
```

`nginx.conf`

```conf
stream {
    upstream rtk2_server {
        least_conn;
        server 192.168.1.1:9345 max_fails=3 fail_timeout=5s;
        server 192.168.1.2:9345 max_fails=3 fail_timeout=5s;
        server 192.168.1.3:9345 max_fails=3 fail_timeout=5s;
   }
   server {
      listen 9345;
      proxy_pass  rtk2_server;
   }
    upstream rtk2_apiserver {
        least_conn;
        server 192.168.1.1:9345 max_fails=3 fail_timeout=5s;
        server 192.168.1.2:9345 max_fails=3 fail_timeout=5s;
        server 192.168.1.3:9345 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     6443;
        proxy_pass rtk2_apiserver;
    }
}
```

## 第一台 Server

```shell
mkdir -p /etc/rancher/rke2/
echo \
"token: rke2-create-token
tls-san:
- lb.server.loacl
- 192.168.1.1
- 192.168.1.3
- 192.168.1.2
- 192.168.1.200
system-default-registry: registry.cn-hangzhou.aliyuncs.com" | \
sudo tee /etc/rancher/rke2/config.yaml > /dev/null

curl -sfL https://rancher-mirror.rancher.cn/rke2/install.sh | INSTALL_RKE2_MIRROR=cn INSTALL_RKE2_VERSION=v1.30.13-rke2r1 sh -

systemctl enable rke2-server

systemctl start rke2-server
```

## 其他 Server

```shell
mkdir -p /etc/rancher/rke2/
echo \
"server: https://192.168.1.1:9345
token: rke2-create-token
tls-san:
- lb.server.loacl
- 192.168.1.1
- 192.168.1.3
- 192.168.1.2
- 192.168.1.200
system-default-registry: registry.cn-hangzhou.aliyuncs.com" | \
sudo tee /etc/rancher/rke2/config.yaml > /dev/null

curl -sfL https://rancher-mirror.rancher.cn/rke2/install.sh | INSTALL_RKE2_MIRROR=cn INSTALL_RKE2_VERSION=v1.30.13-rke2r1 sh -

systemctl enable rke2-server

systemctl start rke2-server

systemctl enable rke2-agent.service

systemctl start rke2-agent.service
```

## Agent

```shell
mkdir -p /etc/rancher/rke2/
echo \
"server: https://lb.server.loacl:9345
token: rke2-create-token
system-default-registry: registry.cn-hangzhou.aliyuncs.com" | \
sudo tee /etc/rancher/rke2/config.yaml > /dev/null

curl -sfL https://rancher-mirror.rancher.cn/rke2/install.sh | INSTALL_RKE2_MIRROR=cn INSTALL_RKE2_VERSION=v1.30.13-rke2r1 INSTALL_RKE2_TYPE="agent" sh -

systemctl enable rke2-agent

systemctl start rke2-agent
```

## 管理

任意 Server 节点

```shell
export PATH=$PATH:/var/lib/rancher/rke2/bin
export CONTAINER_RUNTIME_ENDPOINT=unix:///var/run/k3s/containerd/containerd.sock

cd ~
mkdir .kube
ln -s /etc/rancher/rke2/rke2.yaml .kube/config

kubectl get node
```
