# Kubernetes The Simple Way

## Phase 1 on CONTROL0

```bash
sudo -i
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/01-admin-certificates.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/02-admin-copycerts.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/03-admin-configs.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/04-admin-copyconfigs.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/05-admin-createcopysecret.txt | sudo bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/06-control-etcd.txt | sudo bash

sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem

curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/07-control-kube.txt | sudo bash

kubectl get componentstatuses --kubeconfig /home/azureuser/admin.kubeconfig

curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/08-control0-rbac.txt | sudo bash

RESOURCE_GROUP=$(curl --silent  http://169.254.169.254/Metadata/instance?api-version=2017-08-01 -H metadata:true|jq -r '.compute.resourceGroupName')
EXTERNAL_IP=$(host ${RESOURCE_GROUP}control0.westeurope.cloudapp.azure.com|awk '/has address/ { print $4 }')
curl https://${EXTERNAL_IP}:6443/healthz --insecure
curl --cacert ca.pem https://${EXTERNAL_IP}:6443/version
```

## Phase 2 on WORKER0/WORKER1

```bash
sudo -i
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/10-worker-binaries.txt | sudo bash
# --- wait ---
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/11-worker-install.txt | sudo bash
```

## Phase 6 on CONTROL0

```bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/12-admin-client.txt | sudo bash

# --- wait ---

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# --- wait ---

kubectl get nodes

# DNS

kubectl apply -f https://raw.githubusercontent.com/tvdvoorde/ktsw/master/coredns-mum.yaml
kubectl get pods -l k8s-app=kube-dns -n kube-system
kubectl get pods --all-namespaces

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

```bash
kubectl create secret generic kubernetes-the-hard-way --from-literal="mykey=mydata"
```

on CONTROL0

```bash
sudo ETCDCTL_API=3 etcdctl get --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem /registry/secrets/default/kubernetes-the-hard-way | hexdump -C
```

```bash
RESOURCE_GROUP=$(curl --silent  http://169.254.169.254/Metadata/instance?api-version=2017-08-01 -H metadata:true|jq -r '.compute.resourceGroupName')
EXTERNAL_IP=$(host ${RESOURCE_GROUP}control0.westeurope.cloudapp.azure.com|awk '/has address/ { print $4 }')

  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${EXTERNAL_IP}:6443 \
    --kubeconfig=admin.kubeconfig.remote
  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig.remote
  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig.remote
```

```cmd
scp -i c:\KUBERNETES\private.openssh azureuser@aks3141control0.westeurope.cloudapp.azure.com:/tmp/admin.kubeconfig.remote .
```

curl https://40.119.151.98:6443/healthz --insecure
curl --cacert ca.pem https://40.119.151.98:6443/version


c:\users\ted.van.der.voorde\kubectl get pods --kubeconfig=c:\kubernetes\admin.kubeconfig.remote


c:\users\ted.van.der.voorde\kubectl run mysql --image=mysql:5.6 --env=MYSQL_ROOT_PASSWORD=Welcome1 --port=3306 --restart=Never 
c:\users\ted.van.der.voorde\kubectl expose pod mysql --name=mysql --type=ClusterIP 
c:\users\ted.van.der.voorde\kubectl run wordpress --image=wordpress:4.8-apache --env=WORDPRESS_DB_PASSWORD=Welcome1 --env=WORDPRESS_DB_HOST=mysql.default.svc.cluster.local --port=80 --restart=Never 
c:\users\ted.van.der.voorde\kubectl expose pod wordpress --name=wordpress --type=NodePort 
c:\users\ted.van.der.voorde\kubectl get service -o wide
c:\users\ted.van.der.voorde\kubectl port-forward service/wordpress 8080:80
