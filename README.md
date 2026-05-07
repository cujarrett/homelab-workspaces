# homelab-tenants

XR instance files for all tenant apps running on the [homelab](https://github.com/cujarrett/homelab) cluster.

## Structure

Each top-level directory is a tenant namespace. Files inside are flat — one file per Crossplane XR instance, named `<xr-instance-name>.yaml`. The `kind` in each file tells you which XRD it uses.

```
tenants/
  <namespace>/
    <xr-instance-name>.yaml   ← kind: XSpa | XApi | XWordPressPlatform | …
    <xr-instance-name>.yaml
```

## How deploys work

CI in each source repo calls the reusable workflow at `.github/workflows/update-image-tag.yml` after pushing a new image. The workflow writes the new SHA tag back to the relevant file here. ArgoCD detects the commit and syncs the app automatically.

See [homelab](https://github.com/cujarrett/homelab) for cluster infra, platform compositions, and ArgoCD bootstrap config.

## Adding a new app

1. Create `<namespace>/<xr-instance-name>.yaml` with the XR instance manifest
2. Add the `deploy` job to the source repo's CI:

**Zero-config** — works when the repo name equals the namespace and the XR instance name:

```yaml
deploy:
  needs: build
  uses: cujarrett/homelab-tenants/.github/workflows/update-image-tag.yml@main
  secrets:
    homelab_pat: ${{ secrets.HOMELAB_PAT }}
```

**With namespace override** — when the repo lives in a different namespace (e.g. `my-vinyl-api` belongs to the `my-vinyl` namespace):

```yaml
deploy:
  needs: build
  uses: cujarrett/homelab-tenants/.github/workflows/update-image-tag.yml@main
  with:
    namespace: my-vinyl
  secrets:
    homelab_pat: ${{ secrets.HOMELAB_PAT }}
```

**With full file override** — for repos where the name can't be derived (e.g. a dot in the repo name):

```yaml
deploy:
  needs: build
  uses: cujarrett/homelab-tenants/.github/workflows/update-image-tag.yml@main
  with:
    file: tenants/<namespace>/<xr-instance-name>.yaml
  secrets:
    homelab_pat: ${{ secrets.HOMELAB_PAT }}
```

`HOMELAB_PAT` is a fine-grained PAT scoped to this repo with `Contents: Read and write`.
