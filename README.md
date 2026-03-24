# k8s-lab-argocd

ArgoCD apps repo for k8s lab cluster.

Mirrors the pattern used in production ArgoCD repos.

---

## How it works

ArgoCD watches this repo and deploys applications to the cluster automatically.

```
bootstrap/
  kustomization.yaml          # Entry point — lists all ApplicationSets
  applications/
    nginx-demo.yaml           # ApplicationSet: deploys nginx to matching environments

components/
  nginx-demo/                 # Helm chart wrapper for the app
    Chart.yaml
    values.yaml               # Default values

environments/
  lab/                        # One folder per environment
    apps.yaml                 # Declares env name (used by ApplicationSet generator)
    nginx-demo-values.yaml    # Environment-specific overrides
```

---

## Flow

```
GitHub repo (this)
      ↓
  ArgoCD (running in cluster)
      ↓  reads ApplicationSet from bootstrap/applications/
      ↓  generator scans environments/*/apps.yaml
      ↓  creates one Application per environment found
      ↓  deploys components/<app>/ with env-specific values
      ↓
  Deployed to cluster
```

---

## Bootstrap (first time setup)

1. Install ArgoCD on the cluster:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. Add this repo to ArgoCD (SSH):
   ```bash
   argocd repo add git@github.com-personal:abdalluhmostafa/k8s-lab-argocd.git \
     --ssh-private-key-path ~/.ssh/id_rsa
   ```

3. Apply the bootstrap kustomization:
   ```bash
   kubectl apply -k bootstrap/
   ```

---

## Adding a new app

1. Create `components/<app-name>/` with a Helm chart
2. Create `bootstrap/applications/<app-name>.yaml` as an ApplicationSet
3. Add `environments/lab/<app-name>-values.yaml` for env-specific config
4. Reference the ApplicationSet in `bootstrap/kustomization.yaml`
5. Push → ArgoCD picks it up automatically

## Adding a new environment

1. Create `environments/<env-name>/apps.yaml` with `env: <env-name>`
2. Add `environments/<env-name>/<app>-values.yaml` for each app
3. Push → ApplicationSet generator creates the new Application automatically

---

## Check sync status

```bash
argocd app list
argocd app get nginx-demo-lab
kubectl get applications -n argocd
```
