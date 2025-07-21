# K8s 裸机 LoadBalancer 控制器

裸金属是指 K8s 节点运行在不受限制的 L2 网络（允许控制 ARP），公有云通常不属于裸金属，比如阿里云 AWS

## 环境准备

### 内核参数

`net.ipv4.conf.all.arp_ignore=1`

- 0：默认行为。不论目标IP地址是否位于接收网卡上，都应该对本机IP地址的ARP请求做出响应，包括环回网卡上的地址。
- 1：只对目标IP地址为本地接收网卡上的地址的ARP请求作出响应。
- 2：只对目的IP地址为接收网卡上的本地地址的ARP请求进行响应，且ARP请求的源IP必须与接收网卡处于同一网段。
  
`net.ipv4.conf.all.arp_announce=2`

- 0：默认行为。在默认情况下，系统将使用任意可用的源IP地址作为接口的首选地址。在这种模式下，当发送ARP请求时，内核不会严格限制ARP请求中使用的源IP地址，而是可能选择任何可用的本地地址作为源IP。
- 1：严格模式。尽量避免使用不相关的源IP地址。内核在选择源IP地址时，会尝试使用与目标地址在同一子网内的地址。如果没有这样的地址，它还是会使用接口的主地址。
- 2：最严格模式。必须使用发送接口上的IP地址。内核首先按照arp_announce=1的方式查找，若没有查找到，则强制要求ARP请求使用的源IP地址必须是发送ARP请求的接口的IP地址。如果在这个接口上没有合适的IP地址，则不会发送ARP请求。

```shell
cat <<EOF | sudo tee /etc/sysctl.d/k8s_arp.conf
net.ipv4.conf.all.arp_ignore=1
net.ipv4.conf.all.arp_announce=2

EOF
sudo sysctl --system
```

### kube-proxy

`--ipvs-strict-arp`启用严格ARP，只有本机拥有该 VIP 的网卡会回应 ARP 请求。kube-proxy 会自动配置前者内核参数

### 集群网络

- `eth0`
ip：`10.111.111.2/24`
l2: `switch-a`
- `eth1`
l2: `switch-b`

### 外部网络

- `eth0`
ip：`10.111.111.1/24`
l2: `switch-a`

- `eth1`
ip：`10.112.112.1/24`
l2: `switch-b`

## PureLB 配置

PureLB 是基于主机路由的 Local Routing 模式

### 安装

```shell
helm repo add purelb https://gitlab.com/api/v4/projects/20400619/packages/helm/stable
helm install --create-namespace --namespace=purelb-system purelb purelb/purelb
```

### LBNodeAgent

默认安装后会创建一个 LBNodeAgent

```yaml
apiVersion: purelb.io/v1
kind: LBNodeAgent
metadata:
  name: default
  namespace: purelb-system
spec:
  local:
    extlbint: kube-lb0 # 外部 LB IP 通信网卡
    localint: default  # 本地通信网卡，默认会使用默认路由网卡通信
    sendgarp: false    # EVPN/VXLAN 环境下配置，是否发送 GARP
```

### ServiceGroup

```yaml
apiVersion: purelb.io/v1
kind: ServiceGroup
metadata:
  name: subnet-1
  namespace: purelb-system
spec:
  local:
    v4pools:
    - subnet: 10.111.111.0/24
      pool: 10.111.111-100-10.111.111.200
      aggregation: default
---
apiVersion: purelb.io/v1
kind: ServiceGroup
metadata:
  name: subnet-2
  namespace: purelb-system
spec:
  local:
    v4pools:
    - subnet: 10.112.112.0/24
      pool: 10.112.112.100-10.112.112.200 
      aggregation: default
apiVersion: purelb.io/v1
kind: ServiceGroup
metadata:
  name: subnet-3
  namespace: purelb-system
spec:
  local:
    v4pools:
    - subnet: 10.113.113.0/24
      pool: 10.113.113.100-10.113.113.200 
      aggregation: default      
```

### subnet-1 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    purelb.io/service-group: subnet-1
  labels:
    app: test-nginx
  name: test-nginx
  namespace: test
spec:
  - name: app
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: test-nginx
  type: LoadBalancer
```

本地地址：创建后会分配 `10.111.111.100` VIP，因为 VIP 地址池与 eth0 子网一致，所以会在 eth0 上分配该 VIP

所有子网一致的节点 eth0 都会分配该 VIP，但只有一个节点（通过选举）会应答 VIP 请求，其他节点不会应答

外部节点测试：
`curl 10.111.111.100`

### subnet-2 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    purelb.io/service-group: subnet-2
  labels:
    app: test-nginx
  name: test-nginx
  namespace: test
spec:
  - name: app
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: test-nginx
  type: LoadBalancer
```

本地地址指定网卡：创建后会分配 `10.112.112.100` 地址，需要在 `LBNodeAgent` 中配置 `extlbint: eth1`，这样就会在`eth1`上分配该 VIP，否则会在`kube-lb0`上分配该地址

外部节点测试：
`curl 10.112.112.100`

### subnet-3 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    purelb.io/service-group: subnet-3
  labels:
    app: test-nginx
  name: test-nginx
  namespace: test
spec:
  - name: app
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: test-nginx
  type: LoadBalancer
```

虚拟地址配置：创建后会分配 `10.112.112.100` 地址，因为该 VIP 地址池不属于任何本地子网，默认会在所有节点的`kube-lb0`配置该 VIP，并且每个节点都会应答 VIP 的请求，虚拟地址模式需要有路由通告，该模式适合高性能集群，可通过路由实现负责均衡。

外部节点手动路由配置：`ip route add 10.113.113.0/24 via <NODE_IP>`

外部节点测试：
`curl 10.112.112.100`

## MetalLB L2 配置

MetalLB 使用 ARP 应答模式，会在指定节点上应答 VIP 请求

### 安装

```shell
helm repo add metallb https://metallb.github.io/metallb
helm install --create-namespace --namespace=metallb-system metallb metallb/metallb
```

### IPAddressPool

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: subnet-1
  namespace: metallb-system
spec:
  addresses:
  - 10.111.111.100-10.222.222.200
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: subnet-2
  namespace: metallb-system
spec:
  addresses:
  - 10.112.112.100-10.112.112.200
```

### L2Advertisement

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - subnet-1
  - subnet-2
```

可选配置

```yaml
spec:
  nodeSelectors: # 默认所有节点都可能被挑选出来通告 VIP，这里可以通过标签筛选
  - matchLabels:
      kubernetes.io/hostname: node-2
  - matchLabels:
      kubernetes.io/hostname: node-3
  interfaces: # 默认所有网卡都会应答ARP，这里可以指定部分网卡
  - eth0
  - eth1      
```

### subnet-1 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    metallb.io/address-pool: subnet-1
  labels:
    app: test-nginx
  name: test-nginx
  namespace: test
spec:
  - name: app
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: test-nginx
  type: LoadBalancer
```

外部节点测试：
`curl 10.111.111.100`

### subnet-2 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    metallb.io/address-pool: subnet-2
  labels:
    app: test-nginx
  name: test-nginx
  namespace: test
spec:
  - name: app
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: test-nginx
  type: LoadBalancer
```

因为节点 eth1 没有配置该子网的 IP，所以需要在集群节点上添加一条回程路由`ip route add 10.112.112.0/24 dev eth1`

外部节点测试：
`curl 10.112.112.100`
