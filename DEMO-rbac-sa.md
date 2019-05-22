
On the admin workstation

```
kubectl run curl --image=maiwj/curl --restart=Never -- sleep 3600

# WAIT FOR THE POD TO BE READY

kubectl exec curl -it -- /bin/sh
```

In the bash shell

```
TOKEN=invalidtoken
curl https://kubernetes/api --header "Authorization: Bearer $TOKEN" --insecure
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl https://kubernetes/api --header "Authorization: Bearer $TOKEN" --insecure
curl https://kubernetes/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure
exit
```

On the admin workstation

```
kubectl create serviceaccount podreader
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
kubectl create rolebinding podr-view --role=pod-reader --serviceaccount=default:podreader 
kubectl run curl2 --image=maiwj/curl --serviceaccount=podreader --restart=Never -- sleep 3600 

# WAIT FOR THE POD TO BE READY

kubectl exec curl2 -it -- /bin/sh
```

In the bash shell

```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl https://kubernetes/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure
exit
```
