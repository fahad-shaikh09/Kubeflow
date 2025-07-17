# Kubeflow
Setting up Kubeflow for MLOps pipeline


## Deploying Kubeflow on MacBook with Minikube & Kustomize
**1. Prerequisites**
Minikube installed (brew install minikube)

kubectl (brew install kubectl)

kustomize CLI (brew install kustomize) 


**Sufficient Minikube resources:**
minikube start --cpus=8 --memory=20000
Tip: More CPU/RAM speeds up image pulls and pod startup.

**2. Clone Kubeflow Manifests**
git clone https://github.com/kubeflow/manifests.git
cd manifests

**3. (Optional) Configure Dex credentials**
To change the default login (user@example.com/12341234), generate a bcrypt hash and replace it in:
common/dex/base/dex-passwords.yaml
Then:


kubectl delete secret dex-passwords -n auth
kubectl create secret generic dex-passwords \
  --from-literal=DEX_USER_PASSWORD='<YOUR_HASH>' -n auth
kubectl delete pod -n auth -l app=dex


**4. Install Common Services**
Apply the base kustomize overlays for:

```bash
kustomize build common/cert-manager/cert-manager/base | kubectl apply -f -
kustomize build common/cert-manager/kubeflow-issuer/base | kubectl apply -f -
kustomize build common/istio-*/istio-install/base | kubectl apply -f -
kustomize build common/dex/base | kubectl apply -f -
kustomize build common/oauth2-proxy/base | kubectl apply -f -
kustomize build common/kubeflow-namespace/base | kubectl apply -f -
kustomize build common/kubeflow-roles/base | kubectl apply -f -
```

These install CRDs (for Istio, cert-manager) and set up auth, namespaces, and RBAC.

**5. Launch Kubeflow Stack**
With core services healthy (kubectl get pods -A), deploy all components:

```bash
while ! kustomize build example | kubectl apply -f -; do
  echo "Retrying…"
  sleep 20
done
```

This retries until all CRDs/resources apply cleanly—especially useful for handling timing/order issues. 


**6. Validate Deployment**
Confirm all pods are Running and Ready:


kubectl get pods -A
Look across these key namespaces:

cert-manager

istio-system

auth

kubeflow

**7. Access the Dashboard**
Port‑forward the Istio ingress gateway:
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
Then navigate to http://localhost:8080.

Use default credentials or your custom ones set via Dex.

**8. Explore Kubeflow**
Once logged in, you can:

Launch Jupyter Notebooks in your profile namespace

Upload and run Pipelines (compile with kfp.compiler.Compiler().compile(...))

**Create Katib experiments**

Serve models using KServe InferenceService CRDs

# ✅ Summary of Commands

```bash
minikube start --cpus=8 --memory=20000
git clone https://github.com/kubeflow/manifests.git && cd manifests
```

(Optional) Update Dex password
kubectl delete/create secret steps...

## Install core services
```bash
kustomize build common/cert-manager/cert-manager/base | kubectl apply -f -
kustomize build common/cert-manager/kubeflow-issuer/base | kubectl apply -f -
kustomize build common/istio-*/istio-install/base | kubectl apply -f -
kustomize build common/dex/base | kubectl apply -f -
kustomize build common/oauth2-proxy/base | kubectl apply -f -
kustomize build common/kubeflow-namespace/base | kubectl apply -f -
kustomize build common/kubeflow-roles/base | kubectl apply -f -
```

## Deploy Kubeflow
```bash
while ! kustomize build example | kubectl apply -f -; do
  echo "Retrying apply…"
  sleep 20
done
```

kubectl get pods -A

<img width="1536" height="817" alt="image" src="https://github.com/user-attachments/assets/17b192ef-6e54-4150-ab8f-692e80211187" />


# Access dashboard
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

<img width="1536" height="883" alt="image" src="https://github.com/user-attachments/assets/172fcd1e-cb5f-4892-86e3-fe85074a17e9" />

