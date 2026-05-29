# tor-character-mcp

Kubernetes deploy charts for the TOR character-creation MCP server. Argo-managed via the [argocd-projects](https://github.com/dvystrcil/argocd-projects) app-of-apps.

The build repo is [dvystrcil/tor-character-mcp-docker](https://github.com/dvystrcil/tor-character-mcp-docker). Tracking issue: [dvystrcil/homelab#273](https://github.com/dvystrcil/homelab/issues/273).

## Layout

```
base/
  namespace.yaml            tor-character-mcp namespace (istio ambient)
  harbor-pull-secret.yaml   InfisicalSecret → dockerconfigjson
  deployment.yaml           Single-replica Go binary, scratch image
  service.yaml              ClusterIP :8080
  image-updater-rbac.yaml   Role/RoleBinding so IU can read the pull secret
  kustomization.yaml        Pins the current Harbor tag
image-updater/
  tor-character-mcp-image-updater.yaml   IU CR (writes back to /base)
```

## Image flow

1. PR merges to `tor-character-mcp-docker` → CI builds `:dev` + `:sha-<sha>` and publishes a GitHub release (auto patch-bump via `dvystrcil/release-action@v0.1.1`)
2. `docker-release.yaml` retags Harbor `:dev` → `:X.Y.Z`, `:X.Y`, `:latest`
3. `argocd-image-updater` (the `tor-character-mcp-iu` CR here) detects the new semver tag, writes the new `newTag:` into `base/kustomization.yaml` on `main`
4. Argo CD reconciles, the Deployment rolls

## Routing

Routed by the `gateway-services` chart (in [homelab](https://github.com/dvystrcil/homelab)). HTTPRoute is not in this repo — the chart renders all routes centrally.

## License

[MIT](LICENSE).
