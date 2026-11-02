# SquadRules Helm Charts

Official Helm charts for [SquadRules](https://github.com/SquadRules), published as
versioned OCI artifacts in GitHub Container Registry (GHCR).

| Chart | Source | Purpose |
| --- | --- | --- |
| `mcp` | [charts/mcp](charts/mcp) | SquadRules MCP server, Qdrant, optional Valkey and operator-managed Keycloak/PostgreSQL/Redis |

## Install

```sh
helm install squadrules oci://ghcr.io/squadrules/charts/mcp \
  --version 0.16.0 --namespace squadrules --create-namespace \
  -f my-values.yaml
```

The default chart is an example topology, not a production-ready installation.
Review [configuration and prerequisites](charts/mcp/README.md), especially
embedding-provider credentials, gateway settings, replica placement, and
operator CRDs. The application image is pinned to the published version used by
the [containers](https://github.com/SquadRules/containers) repository; chart
versions are independent of application versions.

For development from this repository:

```sh
helm repo add qdrant https://qdrant.github.io/qdrant-helm
helm repo add valkey https://valkey.io/valkey-helm/
helm dependency build charts/mcp
helm lint charts/mcp --strict
helm template squadrules charts/mcp --namespace squadrules
```

## CI and releases

- Pull requests: strict Helm lint, template/render validation, helm-unittest,
  packaging smoke test, security config scan, and conventional PR-title gate.
- `main`: the same checks continue to run on pushes; security scanning also
  runs on a schedule.
- Releases: changing `charts/mcp/Chart.yaml` on `main` publishes the
  corresponding version to `oci://ghcr.io/squadrules/charts`.
  A published version is immutable in this release process; bump the chart
  version whenever packaging changes. Manual dispatch is available for retrying
  a failed first publication.
- Workflow publishing uses `GITHUB_TOKEN` with `packages: write`; no long-lived
  personal access token is required for publishing in the same organization.

Branch protection currently requires **administrator configuration**: require
the checks `Automation policy passed`, `Integration workflow passed`, and
`Security workflow passed` in the `main` ruleset. After the first successful
GHCR publication, verify that the package visibility permits public pulls.

## Scope and provenance

This repository owns the chart and its release workflow. Cluster operators,
Gateway API controllers, certificate issuance, and secret distribution are
cluster prerequisites—not bundled as operator installers.

Application source lives in [`SquadRules/mcp`](https://github.com/SquadRules/mcp).
