# hotel-booking-gitops

GitOps repo for the hotel-booking microservices app, watched by ArgoCD.

## Structure

- `charts/<service>/` — Helm chart per service (payment-service, booking-service, ui-service)
  - `values.yaml` — base defaults
  - `values-dev.yaml` — dev environment overrides
  - `values-prod.yaml` — prod environment overrides
- `argocd-apps/` — ArgoCD Application manifests, one per service per environment

## First-time setup

Apply all ArgoCD Applications once to register them:

```bash
kubectl apply -f argocd-apps/
```

After that, ArgoCD watches this repo continuously — any commit to `main` that
changes a chart under `charts/<service>/` is auto-synced to the cluster.

## How Jenkins updates a deployment

Each service's Jenkins pipeline updates only its own values file, e.g.:

```bash
sed -i 's|tag:.*|tag: "47"|' charts/payment-service/values-prod.yaml
git commit -am "Deploy payment-service build 47 to prod"
git push
```

## Rollback

```bash
git log --oneline -- charts/payment-service/values-prod.yaml
git revert <bad-commit-sha>
git push origin main
```

ArgoCD will detect the revert and redeploy the previous image automatically.

## Validate a chart locally before pushing

```bash
cd charts/payment-service
helm template . -f values-prod.yaml
```
