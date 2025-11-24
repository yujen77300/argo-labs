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

Run rollout dashboard at http://localhost:3100/rollouts 
```bash
kubectl argo rollouts dashboard -p 3100
```

### Run rollout yaml

```bash
kubectl delete deploy vote
kubectl apply -k staging
kubectl get ro,all
kubectl describe ro vote
```



## Istio Integration (POC)

### Install Istio

Install istioctl (if not already installed):
```bash
# macOS
brew install istioctl

# or manually
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH
```

Install Istio with default profile:
```bash
istioctl install --set profile=default -y
```

Verify Istio installation:
```bash
kubectl get pods -n istio-system
kubectl get svc -n istio-system
```

### Enable Sidecar Injection

Label the staging namespace for automatic sidecar injection:
```bash
kubectl label namespace staging istio-injection=enabled
```

Verify the label:
```bash
kubectl get namespace staging --show-labels
```

### Deploy with Istio

Deploy Istio resources (Gateway and VirtualService):
```bash
kubectl apply -k istio/
```

Deploy application with Canary rollout:
```bash
# Delete existing rollout if any
kubectl delete rollout vote -n staging 2>/dev/null || true

# Apply the Istio-enabled configuration
kubectl apply -f base/rollout-istio.yaml -n staging
kubectl apply -f base/service-stable.yaml -n staging
kubectl apply -f base/service-canary.yaml -n staging
```

Check pods (should see 2 containers per pod: app + Envoy sidecar):
```bash
kubectl get pods -n staging
kubectl get ro,svc,vs,gateway -n staging
```

### Test Canary Deployment

Watch the rollout progress:
```bash
kubectl argo rollouts get rollout vote -n staging --watch
```

Update to a new version to trigger canary:
```bash
kubectl argo rollouts set image vote vote=schoolofdevops/vote:v2 -n staging
```

Check traffic distribution:
```bash
kubectl describe vs vote-vsvc -n staging
```

Manually promote if needed:
```bash
kubectl argo rollouts promote vote -n staging
```

Abort rollout if needed:
```bash
kubectl argo rollouts abort vote -n staging
```

### Access the Application

Get Istio Ingress Gateway external IP/port:
```bash
kubectl get svc istio-ingressgateway -n istio-system
```

For Kind cluster, access via NodePort:
```bash
# Find the NodePort
kubectl get svc istio-ingressgateway -n istio-system -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}'

# Access the app (assuming port 30000-30800 range mapped in kind-cluster.yaml)
curl http://localhost:<nodeport>
```

### Cleanup Istio

Remove Istio resources:
```bash
kubectl delete -k istio/
istioctl uninstall --purge -y
kubectl delete namespace istio-system
kubectl label namespace staging istio-injection-
```