## Get kube contexts
```
kubectl config get-contexts
# Switch to EKS
kubectl config use-context arn:aws:eks:<region>:<account>:cluster/<name>
```

## Deploy ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd \
-f argocd-install.yaml

kubectl get pods -n argocd
# expose ui
kubectl port-forward svc/argocd-server \
-n argocd 8080:443
# get password
kubectl -n argocd \
get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" \
| base64 --decode
```

## Create ArgoCD App
```
kubectl apply -f app.yaml
argocd app list
```