Install ArgoCD:

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