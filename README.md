# vote-deploy

Kubernetes Deployment Code for Vote App

## Prerequisites

- [kind](https://kind.sigs.k8s.io/) installed
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installed

## Setup Instructions

### Build Environment

Create the Kind cluster:
```bash
kind create cluster --config kind-cluster.yaml --name argo-lab
```

Verify cluster information:
```bash
kubectl cluster-info --context kind-argo-lab
kubectl get nodes
```

Check existing clusters:
```bash
kind get clusters
```

To delete the cluster (if needed):
```bash
kind delete cluster --name argo-lab
```

Stop and restart
```bash
docker stop argo-lab-control-plane argo-lab-worker argo-lab-worker2

docker start argo-lab-control-plane
docker start argo-lab-worker argo-lab-worker2

kubectl get nodes #check

```

### Create Namespace

Create staging namespace:
```bash
kubectl create ns staging
```

Set staging as default namespace:
```bash
kubectl config set-context --current --namespace=staging
```

Verify current context:
```bash
kubectl config get-contexts
```

Deploy application:
```bash
kubectl apply -k staging/
```

### Install Argo Rollouts

Install Argo Rollouts Controller and CRDs with ,

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

Validate
```bash
kubectl api-resources | grep -i argo
```

Install argo rollout plugin for kubectl
```bash
brew install argoproj/tap/kubectl-argo-rollouts

# or manually
curl -LO https://github.com/argoproj/argo-rollouts/releases/v.1.8.3/download/kubectl-argo-rollouts-darwin-amd64
chmod +x ./kubectl-argo-rollouts-darwin-amd64
sudo mv ./kubectl-argo-rollouts-darwin-amd64 /usr/local/bin/kubectl-argo-rollouts
```

And validate as,
```bash
kubectl argo rollouts version
```

### Run rollout yaml

```bash
kubectl delete deploy vote
kubectl apply -k staging
kubectl get ro,all
kubectl describe ro vote
```