# homelab-workspaces

XR instance files for all workspace apps running on the [homelab](https://github.com/cujarrett/homelab) cluster.

## Structure

Each top-level directory is a tenant namespace, and every directory becomes one ArgoCD Application automatically. Files inside are flat - one YAML per resource, named after the resource it creates.

```
<namespace>/
  namespace.yaml            ← the Namespace, carrying istio-injection: enabled
  <xr-instance-name>.yaml   ← kind: Spa | Api | Wordpress | …
  <xr-instance-name>.yaml
```

Most files are Crossplane XR instances - the `kind` tells you which XRD. A few directories carry plain Kubernetes manifests alongside them (`launchpad/rbac.yaml`, `sump-pump/sump-pump-bridge-nodeport.yaml`); anything ArgoCD can apply is allowed.

The `namespace.yaml` is not optional. ArgoCD syncs with `CreateNamespace=true` and would make the namespace on its own, but the one it makes has no labels - so the workload never gets an Istio sidecar.

Directories named `guest-<word>-<word>` - `guest-quantum-pickle`, say - are written and deleted by `launchpad-api` through the GitHub API as guest sandboxes come and go. Never hand-edit them. Each one carries a `guest.yaml` holding its creation time and which of the five fixed DNS slots it was given; the slot is what decides its public hostname, and it has nothing to do with the name. Every other directory is hand-maintained.

## How deploys work

CI in each source repo calls the reusable workflow at `.github/workflows/update-image-tag.yml` after pushing a new image. The workflow writes the new SHA tag back to the relevant file here. The `xrs` ApplicationSet in the cluster generates one Application per top-level directory, so ArgoCD picks up the commit and syncs the app - and a brand new directory becomes a new Application with no cluster-side change.

See [homelab](https://github.com/cujarrett/homelab) for cluster infra, platform compositions, and ArgoCD bootstrap config.

## Adding a new app

Create `<namespace>/namespace.yaml` and `<namespace>/<xr-instance-name>.yaml`, then add a `deploy` job to the source repo's CI in one of the three shapes below.

`HOMELAB_WORKSPACES_PAT` holds `homelab-workspaces-deploy`, a fine-grained PAT scoped to this repo with `Contents: Read and write`.

### Zero-config

Works when the repo name equals both the namespace and the XR instance name.

```yaml
deploy:
  needs: build-and-push
  if: github.ref == 'refs/heads/main'
  uses: cujarrett/homelab-workspaces/.github/workflows/update-image-tag.yml@main
  secrets:
    homelab_pat: ${{ secrets.HOMELAB_WORKSPACES_PAT }}
```

### Namespace override

For a repo that lives in a different namespace - `sump-pump-bridge` deploys into `sump-pump`.

```yaml
deploy:
  needs: build-and-push
  if: github.ref == 'refs/heads/main'
  uses: cujarrett/homelab-workspaces/.github/workflows/update-image-tag.yml@main
  with:
    namespace: sump-pump
  secrets:
    homelab_pat: ${{ secrets.HOMELAB_WORKSPACES_PAT }}
```

### File and image override

For a repo whose file name differs from the repo name, or one that builds several images. `platform-connections-demo` needs both: three images out of one tree, each landing in a different file.

```yaml
deploy:
  needs: build-and-push
  if: github.ref == 'refs/heads/main'
  uses: cujarrett/homelab-workspaces/.github/workflows/update-image-tag.yml@main
  with:
    image: ghcr.io/cujarrett/platform-connections-demo-api
    file: platform-connections-demo/upstream-api.yaml
  secrets:
    homelab_pat: ${{ secrets.HOMELAB_WORKSPACES_PAT }}
```

Set `image` whenever the built image is not `ghcr.io/cujarrett/<repo-name>`. The workflow rewrites the line matching `image: <image>:`, so a wrong default silently matches nothing and the deploy passes without changing anything.

## Removing an app

Delete the XR from the cluster first, then the directory. The other order orphans every resource Crossplane composed.

```bash
kubectl delete <kind> <name> -n <namespace>   # Crossplane cascade-deletes composed resources
```

Then remove `<namespace>/` from this repo and push - ArgoCD prunes the Application.
