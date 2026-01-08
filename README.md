# devops-k3s-assessment
# Synthlane DevOps K3s Assessment - OpenWebUI with OIDC

## Candidate Name
Avinash Sharma

## Email
sharma.avi91103@gmail.com

## VM Hostname
sharmaavi91103

---

## Objective
Deploy OpenWebUI on a K3s Kubernetes cluster using Helm and configure authentication via a fake OIDC provider.

---

## Steps Performed

### Install K3s
```bash
curl -sfL https://get.k3s.io | sh -
Create Namespace
bash
Copy code
kubectl create namespace openwebui
Install OpenWebUI
bash
Copy code
helm repo add open-webui https://open-webui.github.io/helm-charts
helm repo update

helm install webui open-webui/open-webui \
--namespace openwebui \
--set service.type=ClusterIP
Create TLS for OIDC
bash
Copy code
mkdir /oidc && cd /oidc
openssl req -x509 -nodes -newkey rsa:2048 \
-keyout tls.key -out tls.crt -days 365 \
-subj "/CN=oidc.local"
Create TLS Secret
bash
Copy code
kubectl create secret tls oidc-cert \
--cert=/oidc/tls.crt \
--key=/oidc/tls.key \
-n openwebui
Apply Fake OIDC
bash
Copy code
kubectl apply -f oidc.yaml
Configure Helm for OIDC
yaml
Copy code
oidc:
  clientId: test
  clientSecret: ""
  issuer: https://fake-oidc.openwebui.svc.cluster.local/.well-known/openid-configuration
  scopes:
    - openid
    - profile
    - email
Upgrade Helm
bash
Copy code
helm upgrade webui open-webui/open-webui -n openwebui -f values-oidc.yaml
kubectl rollout restart statefulset open-webui -n openwebui
Access WebUI
bash
Copy code
kubectl -n openwebui port-forward open-webui-0 8080:8080
Open browser: http://localhost:8080
