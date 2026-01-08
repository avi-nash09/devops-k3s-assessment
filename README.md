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


## Ownership Questions

---

### 1. Production Readiness

**Top 5 risks before going live**

1. Single-node cluster – full outage if VM fails.  
2. No monitoring or alerting – failures unnoticed.  
3. No automated backups – risk of permanent data loss.  
4. Secrets not centrally managed – potential leakage.  
5. No traffic protection or autoscaling.

**First 2 things I would fix**

- Add monitoring & alerting (Prometheus + Alertmanager).  
- Plan node redundancy or managed Kubernetes.

---

### 2. Failure Scenario – Traffic spikes 10x at 2 AM

**What breaks first?**  
Node CPU & memory get saturated → kubelet unresponsive.

**How I recover?**

1. Reboot node from Hetzner console.  
2. Temporarily resize VM.  
3. Scale workloads:

```bash
kubectl scale statefulset open-webui --replicas=3 -n openwebui
What I change next day?

Add HPA.

Add load balancer.

Set resource limits.

3. Security & Secrets
How do I manage secrets?
Kubernetes Secrets + External Secrets Operator.

What must never be in Git?
Private keys, API tokens, DB passwords.

What should be rotated?
OIDC secrets, TLS certs, API keys.

4. Backups & Recovery
What data must be backed up?
PersistentVolumes, Helm configs.

Backup frequency
PV: 6 hours, Config: daily.

How to test recovery?
Restore into staging monthly.

5. Cost Ownership (Hetzner)
How to keep infra cheap?
Small nodes, k3s, no managed services early.

What to avoid early?
Heavy monitoring stacks.

When move away from k3s?

500 users or strict SLA.

Extra Credit – Trust Self-Signed CA
Mount CA cert into container and run:

bash
Copy code
update-ca-certificates
This securely trusts OIDC provider.


