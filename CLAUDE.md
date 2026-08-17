# helm-common

Helm chart library published to `https://mano1233.github.io/helm-common/` via GitHub
Pages. Consumed by the [midgard](https://github.com/mano1233/midgard) cluster.

**This repository is public.** Never commit anything that is not safe to publish.

## Charts

| Chart | Purpose | Consumers |
|---|---|---|
| `n8n` | n8n workflow automation + PostgreSQL subchart + optional nginx webhook-alias proxy | midgard `terraform/modules/n8n` |
| `simple-app` | Generic single-container app (env, secret, configMap, PVC, ingress, probes) | midgard `terraform/modules/{brain,searxng,ha-mcp}` |

## Release Process

`.github/workflows/release.yml` runs `chart-releaser` on push to `main` touching
`charts/**`. It packages the chart and publishes it to the `gh-pages` index.

**Every change to a chart requires a `version` bump in its `Chart.yaml`.**
`chart-releaser` runs with `skip_existing: true`, so an unbumped chart is silently
skipped and consumers keep getting the old package. Use semver:

- patch — template fix, no values change
- minor — new values key, backwards compatible
- major — removed or renamed values key

`appVersion` tracks the upstream application version and is independent of `version`.

After bumping, update the consuming pin in midgard:
`terraform/modules/<service>/variables.tf` → `helm_chart_version`.

## Conventions

- Values are additive and backwards compatible. Renaming or removing a values key
  breaks midgard's Terraform modules at apply time — grep the midgard repo first.
- Guard every optional block with `{{- with }}` or `{{- if }}` so an empty default
  renders nothing rather than an empty YAML key.
- Probes, `resources`, `securityContext`, and `podSecurityContext` are passed through
  verbatim from values. Do not hardcode them in templates.
- Container port is derived from `.Values.service.port`; Services use
  `targetPort: http`. Keep that pairing intact.
- Never put a credential in `values.yaml`, even as a placeholder or example. Secrets
  arrive from Terraform at install time.
- New chart dependencies must be added to the `helm repo add` step in **both**
  `.github/workflows/lint.yml` and `.github/workflows/release.yml`.

## Local Verification

Both CI workflows lint and template every chart. Reproduce locally before pushing:

```bash
helm lint charts/n8n && helm template test charts/n8n --set n8n.encryptionKey=test-key
```

```bash
helm lint charts/simple-app && helm template test charts/simple-app --set image.repository=nginx,image.tag=latest
```

## Git Safety

- NEVER push directly to `main`. Feature branch, then PR.
- NEVER force-push to `main`.
- Branch naming: `DEV-<ticket>-<description>`.
- Scan staged files for tokens, keys, and certificates before committing. This repo is
  public — a leak here is immediately world-readable.
