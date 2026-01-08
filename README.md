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


root@sharmaavi91103:~# kubectl get nodes
kubectl get all -n openwebui
NAME             STATUS   ROLES           AGE   VERSION
sharmaavi91103   Ready    control-plane   34h   v1.34.3+k3s1
NAME                                       READY   STATUS    RESTARTS   AGE
pod/fake-oidc-574c48747c-lwdsl             1/1     Running   0          34h
pod/open-webui-0                           1/1     Running   0          34h
pod/open-webui-ollama-5d99896fd7-jmjlz     1/1     Running   0          34h
pod/open-webui-pipelines-7d8757f9c-4znjm   1/1     Running   0          34h
pod/open-webui-redis-c47dbfbcd-kxkwf       1/1     Running   0          34h

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
service/fake-oidc              ClusterIP   10.43.193.163   <none>        443/TCP     34h
service/open-webui             ClusterIP   10.43.202.73    <none>        80/TCP      34h
service/open-webui-ollama      ClusterIP   10.43.43.78     <none>        11434/TCP   34h
service/open-webui-pipelines   ClusterIP   10.43.199.27    <none>        9099/TCP    34h
service/open-webui-redis       ClusterIP   10.43.27.96     <none>        6379/TCP    34h

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/fake-oidc              1/1     1            1           34h
deployment.apps/open-webui-ollama      1/1     1            1           34h
deployment.apps/open-webui-pipelines   1/1     1            1           34h
deployment.apps/open-webui-redis       1/1     1            1           34h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/fake-oidc-574c48747c             1         1         1       34h
replicaset.apps/open-webui-ollama-5d99896fd7     1         1         1       34h
replicaset.apps/open-webui-pipelines-7d8757f9c   1         1         1       34h
replicaset.apps/open-webui-redis-c47dbfbcd       1         1         1       34h

NAME                          READY   AGE
statefulset.apps/open-webui   1/1     34h
