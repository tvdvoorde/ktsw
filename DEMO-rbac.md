# RBAC Demo

## Requirements

Minikube locally, kubectl running from a commmand prompt

Bash shell

## Instructions

Navigate to the .kube directory in your userprofile folder in a COMMAND prompt

`cd %userprofile%\.kube`

Run

```
kubectl exec -n kube-system -it kube-apiserver-docker-for-desktop cat /run/config/pki/ca.crt > ca.crt
kubectl exec -n kube-system -it kube-apiserver-docker-for-desktop cat /run/config/pki/ca.key > ca.key
```

Start a BASH shell

```
cat > role.yaml <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list","get"]
EOF

cat > binding.yaml <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-developer-binding
subjects:
  - kind: User
    name: dev
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
EOF

openssl genrsa -out dev.key 2048
openssl req -new -key dev.key -subj "/CN=dev" -out dev.csr
openssl x509 -req -in dev.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out dev.crt -days 1000
```

In the command prompt

```
kubectl delete -f role.yaml
kubectl delete -f binding.yaml
```

In Bash

```
curl https://localhost:6445/api/v1/namespaces/default/pods --key dev.key --cert dev.crt --cacert ca.crt --insecure
```

In the command prompt

```
kubectl apply -f role.yaml
kubectl apply -f binding.yaml
```

In Bash

```
curl https://localhost:6445/api/v1/namespaces/default/pods --key dev.key --cert dev.crt --cacert ca.crt --insecure
```

In the command prompt

```
kubectl auth can-i get pods --as dev
kubectl auth can-i create pods --as dev
```
