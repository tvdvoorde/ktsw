# Kubernetes The Simple Way

## Phase 1 on CONTROL0

```bash
sudo -i

curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/01-control.txt | sudo bash
```

or

```bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/01-control-openssl.txt | sudo bash
```

```bash
curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/06-control-etcd.txt | sudo bash

sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem

curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/07-control-kube.txt | sudo bash

kubectl get componentstatuses --kubeconfig /home/azureuser/admin.kubeconfig

curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/08-control-rbac.txt | sudo bash

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

## Phase 3 on CONTROL0

```bash
sudo -i

curl https://raw.githubusercontent.com/tvdvoorde/ktsw/master/12-admin-client.txt | sudo bash

# --- wait ---

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# --- wait ---

kubectl get nodes

# DNS

kubectl apply -f https://raw.githubusercontent.com/tvdvoorde/ktsw/master/coredns15.yaml
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

# --- wait ---

kubectl create secret generic kubernetes-the-hard-way --from-literal="mykey=mydata"

# --- wait ---

# RBAC groups example

cat > role.yaml <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: tester
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list","get"]
EOF

cat > binding.yaml <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: tester-binding
subjects:
  - kind: User
    name: testuser
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: tester
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f role.yaml
kubectl apply -f binding.yaml

openssl genrsa -out testuser.key 2048
openssl req -new -key testuser.key -subj "/CN=testuser" -out testuser.csr
openssl x509 -req -in testuser.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out testuser.crt -days 1000

kubectl get pods --as testuser

kubectl config set-cluster bla \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://127.0.0.1:6443 \
  --kubeconfig=testuser.kubeconfig
kubectl config set-credentials testuser \
  --client-certificate=testuser.crt \
  --client-key=testuser.key \
  --embed-certs=true \
  --kubeconfig=testuser.kubeconfig
kubectl config set-context default \
  --cluster=bla \
  --user=testuser \
  --kubeconfig=testuser.kubeconfig

kubectl get pods --kubeconfig=testuser.kubeconfig



kubectl config use-context default --kubeconfig=admin.kubeconfig


kubectl get pods --server https://127.0.0.1:6443 --certificate-authority=ca.pem --client-certificate=testuser.crt --client-key=testuser.key















sudo ETCDCTL_API=3 etcdctl get --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem /registry/secrets/default/kubernetes-the-hard-way | hexdump -C
```

# --- DEBUGS ---

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

```text
curl https://40.119.151.98:6443/healthz --insecure
curl --cacert ca.pem https://40.119.151.98:6443/version


c:\users\ted.van.der.voorde\kubectl get pods --kubeconfig=c:\kubernetes\admin.kubeconfig.remote


c:\users\ted.van.der.voorde\kubectl run mysql --image=mysql:5.6 --env=MYSQL_ROOT_PASSWORD=Welcome1 --port=3306 --restart=Never 
c:\users\ted.van.der.voorde\kubectl expose pod mysql --name=mysql --type=ClusterIP 
c:\users\ted.van.der.voorde\kubectl run wordpress --image=wordpress:4.8-apache --env=WORDPRESS_DB_PASSWORD=Welcome1 --env=WORDPRESS_DB_HOST=mysql.default.svc.cluster.local --port=80 --restart=Never 
c:\users\ted.van.der.voorde\kubectl expose pod wordpress --name=wordpress --type=NodePort 
c:\users\ted.van.der.voorde\kubectl get service -o wide
c:\users\ted.van.der.voorde\kubectl port-forward service/wordpress 8080:80
```

```text











cat > dns.yaml <<EOF

EOF
