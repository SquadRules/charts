# Optional Kubernetes operator prerequisites

The SquadRules MCP chart does **not** install or manage operators. Install the
specific operator(s) in your cluster before enabling operator-managed resources:

| Chart value | Resource | Required operator |
| --- | --- | --- |
| `redisCluster.enabled` | `RedisFailover` | Spotahome Redis operator |
| `keycloakInstance.enabled` | `Keycloak` | Keycloak Kubernetes operator |
| `postgresCluster.enabled` | `PerconaPGCluster` | Percona PostgreSQL operator |

These CRDs are not required for the default chart, in which the three
operator-managed components are disabled. Configure OLM/operator namespaces,
RBAC and version compatibility according to your cluster distribution and
the vendor's installation guide.

Verify before switching on operator-managed components:

```sh
kubectl get crd redisfailovers.databases.spotahome.com
kubectl get crd keycloaks.k8s.keycloak.org
kubectl get crd perconapgclusters.pgv2.percona.com
```

Only check CRDs for features you enable. Where relevant, enable
`hooks.operatorPrecheckEnabled` to fail installation early if CRDs are missing.

For external services, use `redisCluster.useOwnCluster`,
`keycloakInstance.useOwnCluster` and `postgresCluster.useOwnCluster` and
supply the corresponding `app.*Url` settings.

Gateway API requires a GatewayClass controller; cert-manager is only necessary
when `gateway.tls.certManager.enabled` is enabled. Monitoring custom resources
require Prometheus Operator CRDs. None of these are installed by this chart.

Historical operator and ngrok manifests are archived in the
[original Kairos helm directory](https://github.com/jakub-plichcinski/kairos-mcp/tree/main/helm)
for reference only. They are not maintained as SquadRules operator installers.
