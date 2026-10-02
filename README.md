# SquadRules Helm Charts

This repository hosts the official [Helm](https://helm.sh) charts for
[SquadRules](https://github.com/SquadRules). Charts are packaged and published
as OCI artifacts to the GitHub Container Registry.

## Charts

| Chart | Path | Description |
|---|---|---|
| `mcp` | [`charts/mcp`](charts/mcp) | Deploys the SquadRules MCP server with Qdrant, optional Redis/Valkey, Keycloak and Postgres operator CRs. |

## Usage

Add the OCI registry and install a chart:

```bash
helm install mcp oci://ghcr.io/squadrules/charts/mcp --version <chart-version>
```

Or reference it directly:

```bash
helm upgrade --install mcp oci://ghcr.io/squadrules/charts/mcp \
  --version <chart-version> \
  --namespace squadrules --create-namespace
```

## Chart dependencies

The `mcp` chart depends on the `qdrant` and `valkey` subcharts. Before linting,
templating or packaging locally, add the upstream repositories and build the
dependencies:

```bash
helm repo add qdrant https://qdrant.github.io/qdrant-helm
helm repo add valkey https://valkey.io/valkey-helm/
helm dependency build charts/mcp
```

## Repository layout

```
charts/
  mcp/            # the SquadRules MCP application chart
.github/
  workflows/
    chart-release.yml   # lint, package and publish charts on change
```

## Cluster prerequisites

The `mcp` chart expects certain operators (Keycloak, Percona Postgres, Redis)
and cluster infrastructure (e.g. an ngrok GatewayClass) to be pre-installed
when the corresponding chart values are enabled. Operator and infrastructure
bootstrap manifests live in the application repository
[`SquadRules/mcp`](https://github.com/SquadRules/mcp) under `helm/`, not in this
chart repository.

## Release process

On pushes to `main` that change `charts/mcp/**`, and on a `repository_dispatch`
of type `chart-update`, the [`chart-release`](.github/workflows/chart-release.yml)
workflow lints the chart, packages it, and pushes the packaged artifact to
`oci://ghcr.io/squadrules/charts`.
