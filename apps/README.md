One directory per service:

```
apps/<svc>/
  base/                  # shared manifests
  overlays/sit/          # auto-sync; CI bumps image tag on merge to main
  overlays/uat/          # auto-sync; promote by PR copying the SIT tag
  overlays/production/   # manual sync; promote by PR copying the UAT tag
  preview/               # PR testing envs; tag set per PR (argocd/pr-previews.yaml)
```

Overlay `<env>` becomes Application `<svc>-<env>` in namespace `cugrader-<env>`.
PR preview becomes Application `<svc>-pr-<n>` in namespace `<svc>-pr-<n>`.
