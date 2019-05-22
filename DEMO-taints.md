# Taints & Tolerations & Nodeselector demp

## Requirements

Three node cluster with node names 'worker0', 'worker1' and 'worker2'

## Instructions

```
cat > nginx.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
      nodeSelector:
        conf: fastcpu
      tolerations: 
        - key: "pref"
          operator: "Equal" 
          value: "cpu"
          effect: "NoSchedule"
EOF


cat > httpd.yaml<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: httpd
  name: httpd
spec:
  replicas: 5
  selector:
    matchLabels:
      run: httpd
  template:
    metadata:
      labels:
        run: httpd
    spec:
      containers:
      - image: httpd
        name: httpd
      nodeSelector:
        conf: highmem
      tolerations: 
        - key: "pref"
          operator: "Equal" 
          value: "mem"
          effect: "NoSchedule"
EOF
```

## Label nodes

```
kubectl label node worker1 conf=fastcpu
kubectl label node worker2 conf=highmem
```

## Taint Nodes

```
kubectl taint node worker1 pref=cpu:NoSchedule
kubectl taint node worker2 pref=mem:NoSchedule
```

## Create deployments

```
kubectl create -f nginx.yaml
kubectl create -f httpd.yaml
kubectl run busybox --image=busybox --replicas=5 -- sleep 3600
```

## Observe

```
kubectl get pods -o wide
```

## Clean up

```
kubectl delete -f nginx.yaml
kubectl delete -f httpd.yaml
kubectl delete deployment busybox
kubectl taint node worker1 pref-
kubectl taint node worker2 pref-
kubectl label node worker1 conf-
kubectl label node worker2 conf-


```





