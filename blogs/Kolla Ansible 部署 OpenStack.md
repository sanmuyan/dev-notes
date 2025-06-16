# Kolla Ansible 部署 OpenStack

## 介绍

Kolla-Ansible 使用容器部署 OpenStack 组件，极大简化了部署工作。Kolla-Ansible 可以30分钟部署一个单节点的 OpenStack，同时又可快速部署高可用的生产集群

### 参考文档

[单节点参考文档](https://docs.openstack.org/project-deploy-guide/kolla-ansible/2024.1/quickstart.html)
[多节点参考文档](https://docs.openstack.org/project-deploy-guide/kolla-ansible/2024.1/multinode.html)

## 部署环境

- `OS Ubuntu 22.04`
- `KOLLA_ANSIBLE_VERSION stable/2024.1`
- `deployment host 10.222.222.101`
- `target hosts 10.222.222.102 10.222.222.103 10.222.222.104`

## 准备部署主机

```shell
#apt update
#apt dist-upgrade

apt install git python3-dev libffi-dev gcc libssl-dev
apt install python3-venv
python3 -m venv kolla
source kolla/bin/activate
pip install -U pip
pip install git+https://opendev.org/openstack/kolla-ansible@stable/2024.1
pip install 'ansible-core>=2.15,<2.16.99'

mkdir -p /etc/kolla
chown $USER:$USER /etc/kolla

cp -r kolla/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp kolla/share/kolla-ansible/ansible/inventory/all-in-one .

kolla-ansible install-deps

kolla-genpwd

cat >>/etc/kolla/globals.yml << EOF
kolla_base_distro: "ubuntu"
network_interface: "eth0"
neutron_external_interface: "eth1"
kolla_internal_vip_address: "10.222.222.110"
EOF
```

## 单节点部署

```shell
kolla-ansible -i ./all-in-one bootstrap-servers

kolla-ansible -i ./all-in-one prechecks

kolla-ansible -i ./all-in-one deploy
```

## 多主机部署

```shell
cat >/etc/hosts << EOF
10.222.222.102 control01 network01
10.222.222.103 control02 network02 monitoring01
10.222.222.104 control03 compute01 storage01
EOF

cp kolla/share/kolla-ansible/ansible/inventory/multinode .

kolla-ansible -i ./multinode bootstrap-servers

kolla-ansible -i ./multinode prechecks

kolla-ansible -i ./multinode deploy
```

### 并发问题

因为只有三个节点，所以需要处理 ansible 并发的问题，编辑 multinode 文件，把重叠的节点都注释掉

```shell
[common:children]
control
#network
#compute
#storage
#monitoring
```

## 部署后

```shell
pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/2024.1

kolla-ansible post-deploy

# 获取认证信息
cat /etc/kolla/clouds.yaml
```

### 登录

[Dashboard](https://10.222.222.110)