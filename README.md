# CUGrader-infrastructure

GitOps repo for CUGrader, deployed with [Argo CD](https://argo-cd.readthedocs.io/) (app-of-apps).

## Layout

- `bootstrap/root.yaml` – root Application; apply once by hand.
- `argocd/` – Argo CD config managed by the root app (AppProject, ApplicationSet).
- `apps/<name>/` – one directory per service. Plain manifests, Kustomize, or a Helm chart.
  Each becomes an Application `<name>` in namespace `<name>`.

## Bootstrap

```sh
kubectl create namespace argocd
kubectl apply -n argocd --server-side -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -n argocd -f bootstrap/root.yaml
```

If the repo is private, register it first:
`argocd repo add https://github.com/cu-devclub/CUGrader-infrastructure --username <user> --password <token>`

## Add a service

Create `apps/<name>/` with manifests, push to `main`. Argo CD picks it up automatically.
