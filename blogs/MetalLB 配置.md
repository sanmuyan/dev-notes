# MetalLB 配置

## 安装

kube-proxy 配置

`--ipvs-strict-arp`启用严格ARP，只有本机拥有该 VIP 的网卡会回应 ARP 请求

```shell
helm repo add metallb https://metallb.github.io/metallb
helm install --create-namespace --namespace=metallb-system metallb metallb/metallb
```

## L2 配置

### 集群节点

- `eth0`
ip：`10.111.111.2/24`
- `eth1`

### 外部节点

- `eth0`
ip：`10.111.111.1/24`

- `eth1`
ip：`10.112.112.1/24`

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

创建后会分配 `10.111.111.100` 地址，MetalLB 不会显式在配置该地址，默认会在所有网络接口应答 ARP 广播

外部节点测试：
`curl 10.111.111.100`

### subnet-2 配置

因为节点上没有网卡配置过该网段的地址，默认会在`kube-lb0`配置该地址，因此需要在 `LBNodeAgent` 中配置 `extlbint: eth1`，那样就会在`eth1`上分配该地址

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

创建后会分配 `10.112.112.100` 地址

外部节点测试：
`curl 10.112.112.100`