# Kubernetes The Hard Way on Azure VM's

Adopted from https://github.com/kelseyhightower/kubernetes-the-hard-way

All credits for original content to Kelsey Hightower

## Phase 1 on ADMIN

```
sudo -i
az login --use-device-code
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/01-admin-certificates.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/02-admin-copycerts.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/03-admin-configs.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/04-admin-copyconfigs.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/05-admin-createcopysecret.txt | sudo bash
```

## Phase 2 on CONTROL0/CONTROL1/CONTROL2

```
sudo -i
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/06-control-etcd.txt | sudo bash

# --- wait ---

sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem

# --- wait ---

curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/07-control-kube.txt | sudo bash

kubectl get componentstatuses --kubeconfig /home/azureuser/admin.kubeconfig
```

## Phase 3 only on CONTROL0

```
sudo -i
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/08-control0-rbac.txt | sudo bash
```
## Phase 4 on ADMIN

```
sudo -i
RESOURCE_GROUP=$(curl --silent  http://169.254.169.254/Metadata/instance?api-version=2017-08-01 -H metadata:true|jq -r '.compute.resourceGroupName')
EXTERNAL_IP=$(host ${RESOURCE_GROUP}lbpubip.westeurope.cloudapp.azure.com|awk '/has address/ { print $4 }')
curl https://${EXTERNAL_IP}:6443/healthz --insecure
curl --cacert ca.pem https://${EXTERNAL_IP}:6443/version
```

## Phase 5 on WORKER0/WORKER1/WORKER2

```
sudo -i
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/10-worker-binaries.txt | sudo bash
# --- wait ---
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/11-worker-install.txt | sudo bash
```

## Phase 6 on ADMIN

```
curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/12-admin-client.txt | sudo bash

# --- wait ---

curl https://raw.githubusercontent.com/tvdvoorde/kthw/master/13-admin-routesanddns.txt | sudo bash

# --- wait ---

kubectl get pods -l k8s-app=kube-dns -n kube-system

# --- wait ---

kubectl run nginx --image=nginx --replicas=3
kubectl get pods -o wide

# --- wait ---

kubectl expose deployment nginx --port=80 --type=NodePort
kubectl describe service nginx
curl http://worker0:30131
kubectl delete deployment nginx
kubectl delete service nginx

# --- wait ---

kubectl run httpd --image=httpd --restart=Never
kubectl expose pod httpd --port=80 --type=ClusterIP
kubectl run --generator=run-pod/v1 busybox --image=busybox:1.28 --command -- sleep 3600
kubectl get pods -o wide

# --- wait ---

kubectl exec -ti busybox -- nslookup kubernetes
kubectl exec -ti busybox -- nslookup httpd
kubectl logs httpd
kubectl delete pod httpd
kubectl delete service httpd
kubectl delete pod busybox
kubectl get all 
```

## secrets

```
kubectl create secret generic kubernetes-the-hard-way --from-literal="mykey=mydata"
```

on CONTROL0

```
sudo ETCDCTL_API=3 etcdctl get --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem /registry/secrets/default/kubernetes-the-hard-way | hexdump -C
```
