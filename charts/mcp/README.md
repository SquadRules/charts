# SquadRules MCP Helm chart

A Helm chart for the [SquadRules MCP server](https://github.com/SquadRules/mcp).
It deploys the application and the Qdrant subchart by default, with optional
Valkey, Ollama, Gateway API routes, metrics and operator-managed services.

## Quick start

```sh
helm repo add qdrant https://qdrant.github.io/qdrant-helm
helm repo add valkey https://valkey.io/valkey-helm/
helm dependency build charts/mcp
helm template squadrules charts/mcp --namespace squadrules
```

To install the published OCI chart, choose a released chart version:

```sh
helm upgrade --install squadrules oci://ghcr.io/squadrules/charts/mcp \
  --version 0.16.0 -n squadrules --create-namespace -f my-values.yaml
```

The `0.16.0` example is the target version for this change; verify publication
in the repository Actions tab before installing from GHCR.

## Required configuration

The example `values.yaml` is deliberately not a fully configured deployment.
The default topology requests two MCP replicas and three Qdrant replicas;
`ha.antiAffinity.mode: hard` can prevent scheduling on small clusters.
Override to `soft` or `none` for a single-node test cluster.

1. Configure embeddings. Use `app.embedding.openai.existingSecret` for an
   existing Secret containing `OPENAI_API_KEY`, or configure your own embedding
   provider through `app.extraEnv` (for example `TEI_BASE_URL` and `TEI_MODEL`).
   Never put API keys in committed values files.
2. Set `app.qdrantUrl` when the application should use the chart's Qdrant
   service or your external instance. See rendered service names.
3. Set `gateway.hostname` and `gateway.gatewayClassName` (and TLS as required)
   if using Gateway API. `gateway.createGateway` defaults to false; supply an
   existing Gateway or enable creation. Template-only rendering cannot discover
   cluster GatewayClass objects via `lookup`.
4. When enabling OIDC, set `app.auth.enabled`,
   `app.auth.callbackBaseUrl`, `app.keycloakUrl` and
   `app.keycloakInternalUrl` as appropriate. The bundled realm
   `files/squadrules-realm.json` defines the `squadrules-mcp` and
   `squadrules-cli` clients. Keycloak realm import requires the operator
   and a managed Keycloak instance.
5. `credentials.autoGenerate` produces an internal Secret using an install
   hook. For declarative/GitOps deployments, consider
   `credentials.existingSecret` or `app.existingSecret` and provision Secrets
   independently. Do not treat auto-generated credentials as a backup strategy.

Example values (illustrative addresses; replace with your own):

```yaml
app:
  image:
    repository: quay.io/squadrules/mcp
    tag: "5.14.3"
  qdrantUrl: http://squadrules-qdrant:6333
  embedding:
    openai:
      existingSecret: squadrules-openai
gateway:
  enabled: false
```

## Optional operator CRs

Redis, Keycloak and Percona PostgreSQL operators are **not** installed by
this chart. See [operator prerequisites](docs/OPERATORS.md). Switch on
`redisCluster.enabled`, `keycloakInstance.enabled` or
`postgresCluster.enabled` only after installing their CRDs and configuring
the operator namespace permissions. Use `*.useOwnCluster: true` and the
`app.*Url` settings for external clusters.

## Monitoring

ServiceMonitor and PrometheusRule resources require Prometheus Operator CRDs.
Enable the relevant `monitoring.serviceMonitor.*.enabled` switches. The
default alerts reference `squadrules_http_requests_total`, the metric
exported by the current application. Custom rules can be supplied under
`monitoring.prometheusRule.rules`.

## Tests

```sh
helm lint charts/mcp --strict
helm template squadrules charts/mcp --namespace squadrules
helm unittest charts/mcp   # helm-unittest plugin required
helm package charts/mcp -d dist
```

The CI workflows perform these checks on pull requests; the OCI publish
workflow only runs for trusted `main` events. Chart version and image version
are separate: a chart release can deploy a fixed version of SquadRules MCP.
