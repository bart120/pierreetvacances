## 1. Vérifier le contexte AKS

```bash
kubectl config current-context
kubectl get nodes
```

## 2. Tester Helm localement

```bash
helm lint ./helm/demo-app
helm template demo-app ./helm/demo-app
```

## 3. Déployer manuellement avec Helm

```bash
kubectl create namespace demo-helm

helm upgrade --install demo-app ./helm/demo-app \
  -n demo-helm \
  --set image.repository=registry.gitlab.com/mon-groupe/aks-day3-live-demo/demo-app \
  --set image.tag=latest
```

```bash
kubectl get pods -n demo-helm
kubectl get svc -n demo-helm
kubectl get ingress -n demo-helm
```

## 4. Installer Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
```

Accès local :

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Mot de passe admin :

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

URL : https://localhost:8080

## 5. Créer le namespace GitOps

```bash
kubectl create namespace demo-gitops
```

Dans Argo CD :

- Application name : `demo-app`
- Project : `default`
- Repository URL : URL du dépôt GitLab
- Revision : `main`
- Path : `helm/demo-app`
- Cluster : `https://kubernetes.default.svc`
- Namespace : `demo-gitops`
- Sync policy : `Automatic`

## 6. Démontrer le GitOps

Modifier `helm/demo-app/values.yaml` :

```yaml
replicaCount: 2
```

Puis :

```bash
git add .
git commit -m "scale demo app to 2 replicas"
git push
```

Vérifier :

```bash
kubectl get pods -n demo-gitops
```

## 7. Démontrer la dérive

```bash
kubectl scale deployment demo-app-demo-app -n demo-gitops --replicas=1
kubectl get pods -n demo-gitops
```

Argo CD doit remettre l'état défini dans Git.

## 8. Installer Prometheus / Grafana

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring

helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring
```

```bash
kubectl get pods -n monitoring
```

Accès Grafana :

```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

URL : http://localhost:3000

Identifiants par défaut :

```text
admin / prom-operator
```

## 9. Nettoyage

```bash
helm uninstall demo-app -n demo-helm
kubectl delete namespace demo-helm
kubectl delete namespace demo-gitops
kubectl delete namespace argocd
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```
