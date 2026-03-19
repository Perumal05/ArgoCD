----------------------------------------------------------------------------------------------------------------------------------
Install ArgoCD:
----------------------------------------------------------------------------------------------------------------------------------

kubectl create namespace argocd
kubectl create -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/crds.yaml
kubectl create -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Check Installation:

kubectl get pods -n argocd

Expose ArgoCD UI:

kubectl port-forward svc/argocd-server -n argocd 8080:443
https://localhost:8080

UI Login:

Username: admin
Password: 

kubectl get secret argocd-initial-admin-secret `
  -n argocd `
  -o jsonpath="{.data.password}" |
  ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }

  Update the password on UI

  ![alt text](argo-pass.png)

----------------------------------------------------------------------------------------------------------------------------------
Install Vault with Helm:
----------------------------------------------------------------------------------------------------------------------------------

kubectl create namespace vault

helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

helm install vault hashicorp/vault --set server.dev.enabled=true --namespace vault 

kubectl get pods -n vault
 
kubectl port-forward svc/vault 8200:8200 -n vault

✅ 1. If you used dev mode (easy case)

Token: "root"

----------------------------------------------------------------------------------------------------------------------------------
🔷 🔐 Step 2: Add Secrets in Vault
----------------------------------------------------------------------------------------------------------------------------------

Open Vault UI → go to Secrets Engine

Enable KV Engine

Path: secret/

Version: KV v2

Add Secrets
👉 App Secret

Path:

secret/data/java-app

Key:

password = my-db-password
👉 DockerHub Secret

Path:

secret/data/dockerhub

Keys:

username = YOUR_USERNAME
password = YOUR_PASSWORD

----------------------------------------------------------------------------------------------------------------------------------
Install External Secrets Operator:
----------------------------------------------------------------------------------------------------------------------------------

kubectl create namespace external-secrets

helm repo add external-secrets https://charts.external-secrets.io
helm repo update

helm install external-secrets `
  external-secrets/external-secrets `
  -n external-secrets

kubectl get pods -n external-secrets

Connect ESO to Vault:

🔑 Create Vault Token Secret:

kubectl create secret generic vault-token `
  --from-literal=token=root `
  -n external-secrets

apiVersion: external-secrets.io/v
kind: ClusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "http://vault.vault.svc.cluster.local:8200"
      path: "secret"
      version: "v2"
      auth:
        tokenSecretRef:
          name: vault-token
          key: token
          namespace: external-secrets


kubectl apply -f clustersecretstore.yaml

----------------------------------------------------------------------------------------------------------------------------------

Then configure your respective external secrets with exact path and varaibles that you defined in the hashicrop vault