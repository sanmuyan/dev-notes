# PureLB 配置

## 安装

节点内核配置

`net.ipv4.conf.all.arp_ignore=1`

- 0：不论目标IP地址是否位于接收网卡上，都应该对本机IP地址的ARP请求做出响应，包括环回网卡上的地址。
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

kube-proxy 配置

`--ipvs-strict-arp`启用严格ARP，只有本机拥有该 VIP 的网卡会回应 ARP 请求

```shell
helm repo add purelb https://gitlab.com/api/v4/projects/20400619/packages/helm/stable
helm repo update
helm install --create-namespace --namespace=purelb purelb purelb/purelb
```

## 配置

### 集群节点

- `eth0`
ip：`10.111.111.2/24`
- `eth1`

### 外部节点

- `eth0`
ip：`10.111.111.1/24`

- `eth1`
ip：`10.112.112.1/24`

### LBNodeAgent

默认安装后会创建一个 LBNodeAgent

```yaml
apiVersion: purelb.io/v1
kind: LBNodeAgent
metadata:
  name: default
  namespace: purelb
spec:
  local:
    extlbint: kube-lb0 # 外部 LB IP 通信网卡
    localint: default  # 本地通信网卡，默认会使用默认路由网卡通信
    sendgarp: false
```

### ServiceGroup

```yaml
apiVersion: purelb.io/v1
kind: ServiceGroup
metadata:
  name: subnet-1
  namespace: purelb
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
  namespace: purelb
spec:
  local:
    v4pools:
    - subnet: 10.112.112.0/24
      pool: 10.112.112.100-10.112.112.200 
      aggregation: default
---      
apiVersion: purelb.io/v1
kind: ServiceGroup
metadata:
  name: subnet-3
  namespace: purelb
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

创建后会分配 `10.111.111.100` 地址，因为节点的`eth0`网卡配置了`10.111.111.2`，因此`10.111.111.100` 会配置在`eth0`上

外部节点测试：
`curl 10.111.111.100`

### subnet-2 配置

因为节点上没有网卡配置过该网段的地址，默认会在`kube-lb0`配置该地址，因此需要在 `LBNodeAgent` 中配置 `extlbint: eth1`，那样就会在`eth1`上分配该地址

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

创建后会分配 `10.112.112.100` 地址

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

创建后会分配`10.113.113.100`，因为外部节点没有配置过该子网内的地址，所以需要在外部节点添加路由：

- `extlbint: eth1` `ip route add 10.113.113.0/24 dev eth1`
- `extlbint: kube-lb0` `ip route add 10.113.113.0/24 via 10.111.111.2`

外部节点测试：
`curl 10.113.113.100`
