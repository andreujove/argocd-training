# ArgoCD Training: Complete Setup Guide 🚀

This repository provides a single-file, comprehensive walkthrough to deploy and access **ArgoCD** on a local Kubernetes cluster.

---

## 📋 Prerequisites
* **Minikube:** v1.33.0
* **Kubectl:** v1.34.1
* **OS:** Linux/macOS (for shell commands)

---

## 🛠️ Full Implementation & Deployment

Follow this unified process to initialize your environment, install the management tools, and deploy the ArgoCD stack.


### 1. INITIALIZE CLUSTER & CLI. Start Minikube with optimized resources and install the ArgoCD CLI binary
```bash
minikube start -p test-argocd --kubernetes-version=v1.33.0 --cpus=2 --memory=4098
curl -sSL -o argocd-linux-amd64 [https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64](https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64)
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

### 2.1 DEPLOY ARGOCD STACK USING HELM. Create the namespace and apply manifests using server-side apply to avoid conflicts
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm search repo argo/argo-cd -l
helm show values argo/argo-cd --version 9.4.7 > values.yaml
helm upgrade --install argocd argo/argo-cd \
  --version 9.4.7 \
  --namespace argocd \
  --create-namespace \
  -f values.yaml
helm list -n argocd
kubectl port-forward service/argocd-server -n argocd 8080:443
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 2.2 DEPLOY ARGOCD USING MANIFESTS DIRECTLY
```bash
kubectl apply -n argocd --server-side --force-conflicts -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
# Wait for the server deployment to be fully available before proceeding
kubectl wait --for=condition=available --timeout=90s deployment/argocd-server -n argocd
```

### 3. ACCESS & AUTHENTICATION. Start a background tunnel, verify the redirect, and extract the admin password
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 > /dev/null 2>&1 &
curl -ILk [http://127.0.0.1:8080/](http://127.0.0.1:8080/)
echo "ArgoCD Admin Password:"
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

### 4. Install resources:
```bash
kubectl apply -f argo-resources/app-no-helm.yaml
```



