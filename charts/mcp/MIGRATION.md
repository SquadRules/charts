# Migration from Kairos Helm installations

The chart defaults were rebranded for **new** SquadRules releases. This is
not a drop-in rename of existing Kubernetes resources.

## Existing release: retain immutable identities

A Kubernetes Deployment's `spec.selector` cannot be changed in place.
Preserve the original app selector and release-managed resource names when
upgrading a Kairos deployment, or plan a separate migration with downtime
and data backups.

Example compatibility overrides for a previous `kairos` release:

```yaml
global:
  name: kairos
  hostname: kairos.example.com
app:
  name: kairos-mcp
  auth:
    realm: kairos
    clientId: kairos-mcp
gateway:
  gatewayName: kairos-gateway
redisCluster:
  name: kairos-redis
postgresCluster:
  name: kairos-postgres
keycloakRealmImport:
  name: kairos-realm-import
  realmFile: kairos-realm.json
keycloakDcrCleanup:
  realms: [kairos]
  protectedClients: [kairos-mcp, kairos-cli]
```

Install using the **same Helm release name, namespace, and values used
previously**. Audit generated names and `helm diff` before proceeding; the
example is not a substitute for checking your running installation.

The old `files/kairos-realm.json` is retained for explicit compatibility.
Keycloak realm/client identifiers are external identity contracts. Never
silently rename or re-import an active realm. Likewise, preserve existing
Secrets: the chart's secret-resolution helper intentionally recognizes
`kairos-mcp-credentials` from older installations.

## New SquadRules deployments

New chart defaults use `squadrules-mcp`, `squadrules-gateway`, and the
optional `squadrules` Keycloak realm and its clients. The default image
is supplied by [SquadRules/containers](https://github.com/SquadRules/containers),
not the old Kairos image.

The bundled default alert names were renamed and the HTTP error-rate metric
was fixed to `squadrules_http_requests_total`. If external Alertmanager
routing or dashboards use old names, update them intentionally. Custom
`monitoring.prometheusRule.rules` are unaffected.

Do not assume the chart automatically migrates Qdrant vectors, Redis data,
PostgreSQL volumes, OIDC clients, DNS names or credentials. Plan those separately.
