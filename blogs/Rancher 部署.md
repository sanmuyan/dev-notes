# Rancher 部署

## Helm 方式

```shell
helm repo add jetstack https://charts.jetstack.io
helm repo update

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.crds.yaml
  
helm install cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--version v1.18.2

helm repo add rancher-latest https://releases.rancher.com/server-charts/latest  
  
kubectl create namespace cattle-system  

helm install rancher rancher-latest/rancher \  
--namespace cattle-system \  
--set hostname=<IP_OF_LINUX_NODE>.sslip.io \  
--set replicas=1 \  
--set bootstrapPassword=<PASSWORD_FOR_RANCHER_ADMIN> \
--version 2.12.0
```