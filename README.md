# homelab-tenants

XR instance files for all tenant apps running on the [homelab](https://github.com/cujarrett/homelab) cluster.

## What lives here

Each directory under `tenants/` is an ArgoCD-managed application containing Crossplane XR instance files for a single tenant.

| Tenant | App |
|---|---|
| `js-pollock/` | js-pollock SPA |
| `mattjarrett-com/` | mattjarrett.com WordPress |
| `mattjarrett-dev/` | mattjarrett.dev SPA |
| `my-vinyl/` | my-vinyl SPA + API |
| `sump-pump/` | sump-pump bridge, consumer, and supporting resources |

## How deploys work

CI in each source repo calls the reusable workflow at `.github/workflows/update-image-tag.yml` after pushing a new image. The workflow writes the new SHA tag back to the relevant file here. ArgoCD detects the commit and syncs the app automatically.

See [homelab](https://github.com/cujarrett/homelab) for cluster infra, platform compositions, and ArgoCD bootstrap config.

## Adding a new tenant

1. Create `tenants/<app>/` with the XR instance file(s) following the naming convention (`spa.yaml` or `api.yaml`)
2. Add the zero-config `deploy` job to the source repo's CI:

```yaml
deploy:
  needs: build
  uses: cujarrett/homelab-tenants/.github/workflows/update-image-tag.yml@main
  secrets:
    homelab_pat: ${{ secrets.HOMELAB_PAT }}
```

If the repo name doesn't match the namespace convention, add a `file:` override.
