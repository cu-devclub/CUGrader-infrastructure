# CUGrader-infrastructure

GitOps repo for CUGrader, deployed with [Argo CD](https://argo-cd.readthedocs.io/) (app-of-apps).

## Layout

- `bootstrap/root.yaml` – root Application; apply once by hand.
- `argocd/` – Argo CD config managed by the root app:
  - `envs-auto.yaml` – SIT and UAT, auto-sync.
  - `envs-production.yaml` – production, manual sync.
  - `pr-previews.yaml` – one env per service-repo PR labelled `preview`.
- `apps/<svc>/` – one directory per service (`gateway` = APISIX, `user` = Go Gin). See `apps/README.md`.

## Release flow

1. PR in a service repo with label `preview` → CI pushes `ghcr.io/cu-devclub/cugrader-<svc>:<sha>` → Argo CD deploys `<svc>-pr-<n>`. Gateway previews are served at `https://<head short sha>-gateway-grader-sit.sittha.net`.
2. Merge to `main` → CI pushes image and bumps `newTag` in `apps/<svc>/overlays/sit/` → SIT auto-syncs.
3. PR here copying the tag into `overlays/uat/` → UAT auto-syncs.
4. PR here copying the tag into `overlays/production/` → click Sync in Argo CD.

## Bootstrap

```sh
kubectl create namespace argocd
kubectl apply -n argocd --server-side -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -n argocd -f bootstrap/root.yaml
```

If the repo is private, register it first:
`argocd repo add https://github.com/cu-devclub/CUGrader-infrastructure --username <user> --password <token>`

## Add a service

Copy `apps/user/` to `apps/<svc>/`, rename, push to `main`. For PR envs, add the repo to the `list` in `argocd/pr-previews.yaml`.

PR envs need a GitHub token secret:
`kubectl -n argocd create secret generic github-token --from-literal=token=<PAT>`

## Preview URLs

`*.sittha.net` reaches the cluster through a Cloudflare Tunnel:
- DNS: CNAME `*` → `<tunnel-id>.cfargotunnel.com` (proxied).
- Tunnel public hostname: `*.sittha.net` → the ingress controller's HTTP service.

A service gets a URL only if its `preview/` has an Ingress. Today only `gateway` has one. Backend services stay private so their `/private` endpoints are never exposed.
