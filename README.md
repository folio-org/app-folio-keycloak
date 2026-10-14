# app-eureka-infrastructure

Virtual application descriptor declaring the infrastructure capabilities required by Eureka: PostgreSQL, Kafka and OpenSearch.

## Modules

This application is a **virtual application**: it has no backend modules and no UI modules. It exists only as
metadata for registration, entitlement state and dependency validation.

## Dependencies

| Application  | Version    | Optional |
|:-------------|:-----------|:---------|
| `postgres`   | `>=16.0`   | no       |
| `kafka`      | `>=3.5.1`  | no       |
| `opensearch` | `>=2.19.0` | no       |

All dependencies are required (`"optional": false`): entitlement is blocked if any of them is unavailable.

Names are **concrete implementations** (`opensearch`), not abstract capabilities (such as `search-engine`).

## Virtual applications

A virtual application is a module-less application descriptor that represents an infrastructure capability. It uses the
existing FOLIO application descriptor schema managed by `mgr-applications`; there is no separate schema, registry or API.

Entitling a virtual application only establishes dependency satisfaction state. It does **not**:

- deploy or health-check infrastructure;
- run module tenant APIs or create discovery records;
- create Kong or Keycloak resources;
- publish module lifecycle events or distribute runtime connection configuration.

Virtual applications never appear in module discovery, bootstrap, sidecar routing or Okapi module registration.

### Order of operations

1. Register the virtual application descriptors (`postgres`, `kafka`, `opensearch`).
2. Establish virtual availability for the tenant/environment (entitle the virtual applications).
3. Entitle the standard applications that depend on them.

Required virtual dependencies that are missing, unresolved, version-incompatible or unavailable block entitlement, upgrade
and desired-state workflows. Revoking a virtual application is rejected while installed applications still require it,
unless those applications are removed in the same workflow.

### Declaring a dependency from another application

Application teams add the virtual application to the `dependencies` array of their `application.template.json`:

```json
{
  "dependencies": [
    { "name": "kafka", "version": ">=3.5.1" },
    { "name": "opensearch", "version": ">=2.19.0", "optional": true }
  ]
}
```

`optional` is written to the descriptor only when present in the template and requires `folio-application-generator`
`1.5.0-SNAPSHOT` or later.

### Platform descriptor overlap

Infrastructure metadata in the platform descriptor remains provisioning/platform metadata. Duplication with these virtual
application descriptors is accepted for now.
