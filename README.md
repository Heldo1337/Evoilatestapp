# Evoila Test App

## Vorgabe
- Kubernetes-Cluster aufsetzen
- ArgoCD deployen
- Monitoring-Stack mit Prometheus, Grafana und Loki einrichten
- Beispielapplikation

## Schritt 1 — Voraussetzungen
- Docker Desktop installiert
- WSL2 mit Ubuntu

## Schritt 2 — k3d installieren
```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

## Schritt 3 — Cluster erstellen
```bash
k3d cluster create testcluster --servers 1 --agents 2
kubectl get nodes
```
→ 1 Control Plane + 2 Worker, alle Ready

## Schritt 4 — Namespaces anlegen
```bash
kubectl create namespace dev
kubectl create namespace monitoring-dev
```

## Schritt 5 — Test-Pod
```bash
kubectl run test-app --image=nginx --restart=Never -n dev --labels="app=test-app"
```

## Schritt 6 — Prometheus
```bash
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
```

## Schritt 7 — Grafana
```bash
kubectl apply -f grafana-config.yaml
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
```

## Schritt 8 — ArgoCD installieren
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml --server-side --force-conflicts
```

## Schritt 9 — ArgoCD Zugang
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Schritt 10 — Eigene App bauen
```bash
docker build -t heldo1337/hello-kevin:v1 ./app
docker login
docker push heldo1337/hello-kevin:v1
```

## Schritt 11 — App manuell testen
```bash
kubectl apply -f hello-kevin-deployment.yaml
kubectl port-forward -n dev svc/hello-kevin-service 5000:5000
```
→ http://localhost:5000 lief

## Schritt 12 — Manuellen Test entfernen
```bash
kubectl delete -f hello-kevin-deployment.yaml
```

## Schritt 13 — ArgoCD Application anlegen
```bash
kubectl apply -f argocd/argocd-app.yaml
```
→ ArgoCD übernimmt jetzt das Deployment automatisch aus dem Repo

## Schritt 14 — Node Exporter + kube-state-metrics
```bash
kubectl apply -f Node_exporter.yaml
kubectl apply -f Kube-state-metrics.yaml
```

## Schritt 15 — Prometheus Config erweitern
Scrape-Jobs für node-exporter und kube-state-metrics ergänzt, dann:
```bash
kubectl apply -f prometheus-config.yaml
kubectl rollout restart deployment -n monitoring-dev
```

## Schritt 16 — Grafana Dashboard
Kleines Dashboard importiert mit CPU, Memory, Running Pods, Ready Replicas, Nodes Ready

## Ergebnis
```bash
kubectl get pods -n argocd
kubectl get pods -n monitoring-dev
kubectl get pods -n dev
```
→ alles Running, ArgoCD App Healthy + Synced, Prometheus Targets UP
