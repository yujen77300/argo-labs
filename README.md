# vote-deploy
Kubernetes Deployment Code for Vote App


# command

build enviroment
```bash
kind create cluster --config kind-cluster.yaml --name argo-lab
kubectl cluster-info --context kind-argo-lab => cluster 的基本資訊
kubectl get nodes
kind get clusters
kind delete cluster --name argo-lab
```


create ns
```bash
kubectl create ns staging
kubectl config set-context --current --namespace=staging
kubectl config get-contexts
kubectl apply -k staging/
```