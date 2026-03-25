# Radius Resource Type Catalog

## Namespaces

### Applications.Core (Built-in)

These are built into Radius and available without registration.

| Type | API Version | Description | Recipe-based? |
|------|------------|-------------|---------------|
| `Applications.Core/applications` | `2023-10-01-preview` | Application grouping | No |
| `Applications.Core/containers` | `2023-10-01-preview` | Container workloads | **No** (directly managed) |
| `Applications.Core/gateways` | `2023-10-01-preview` | HTTP ingress gateways | No |
| `Applications.Core/volumes` | `2023-10-01-preview` | Persistent volumes | Yes |
| `Applications.Core/environments` | `2023-10-01-preview` | Environments | No |

### Applications.Datastores (Built-in)

| Type | API Version | Description |
|------|------------|-------------|
| `Applications.Datastores/redisCaches` | `2023-10-01-preview` | Redis caches |
| `Applications.Datastores/sqlDatabases` | `2023-10-01-preview` | SQL databases |
| `Applications.Datastores/mongoDatabases` | `2023-10-01-preview` | MongoDB databases |

### Radius.Compute (radius-resource-types)

Must be registered. **All are recipe-based.**

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.Compute/containers` | `2025-08-01-preview` | Container workloads (recipe-based) |
| `Radius.Compute/persistentVolumes` | `2025-08-01-preview` | Persistent volumes |
| `Radius.Compute/routes` | `2025-08-01-preview` | HTTP routing (requires Gateway API) |

### Radius.Data (radius-resource-types)

Must be registered. All are recipe-based.

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.Data/postgreSqlDatabases` | `2025-08-01-preview` | PostgreSQL databases |
| `Radius.Data/mySqlDatabases` | `2025-08-01-preview` | MySQL databases |

### Radius.Security (radius-resource-types)

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.Security/secrets` | `2025-08-01-preview` | Secret stores |

### Radius.Storage (radius-resource-types)

Must be registered. All are recipe-based.

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.Storage/blobStorages` | `2025-08-01-preview` | Blob storage |

### Radius.AI (radius-resource-types)

Must be registered. All are recipe-based.

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.AI/agents` | `2025-08-01-preview` | LLM-powered AI agents (Azure OpenAI + AI Search + observability) |

## Common Properties

All resource types share these properties:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `environment` | string | Yes | Radius Environment ID |
| `application` | string | No | Radius Application ID |

## Radius.Data/postgreSqlDatabases Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `size` | `'S' \| 'M' \| 'L'` | No | Database instance size (defaults to `S`) |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `host` | string | Database server hostname |
| `port` | string | Port number |
| `database` | string | Database name |
| `username` | string | Admin username |
| `password` | string | Admin password |

## Radius.Data/mySqlDatabases Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `database` | string | No | Database name (defaults to `application-name`) |
| `username` | string | No | Database username (defaults to `application-name-user`) |
| `version` | string | No | MySQL server version in X.Y format (defaults to `8.4`) |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `host` | string | Database hostname |
| `port` | integer | Database port |
| `database` | string | Database name |
| `username` | string | Database username |
| `password` | string | Database password |

## Radius.Compute/containers Schema

### Input Properties

| Property | Type | Description |
|----------|------|-------------|
| `containers` | map of container objects | Container definitions (keyed by name) |
| `connections` | map of connection objects | Connections to other resources |

Each container object:

| Property | Type | Description |
|----------|------|-------------|
| `image` | string | Container image reference |
| `ports` | map of port objects | Port mappings |
| `env` | map of strings | Environment variables |
| `readinessProbe` | probe object | Readiness probe configuration |
| `livenessProbe` | probe object | Liveness probe configuration |

## Radius.Compute/persistentVolumes Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `sizeInGib` | integer | Yes | Size in gibibytes of the persistent volume |
| `allowedAccessModes` | string | No | Access mode restriction (defaults to `ReadWriteOnce`) |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `claimName` | string | Normalized PersistentVolumeClaim name created by the recipe |

## Radius.Compute/routes Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `kind` | string | Yes | Route kind: `HTTPRoute`, `TCPRoute`, `TLSRoute`, or `UDPRoute` |
| `hostnames` | array of strings | No | Hostnames for the route |
| `rules` | array of rule objects | Yes | Routing rules with matches and destination containers |
| `gatewayName` | string | Yes | Name of the Gateway resource the routes attach to |
| `gatewayNamespace` | string | Yes | Namespace where the Gateway exists |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `listener.hostname` | string | Hostname of the listener |
| `listener.port` | integer | Port of the listener |
| `listener.protocol` | string | Protocol of the listener |

## Radius.Security/secrets Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `kind` | string | No | Kind of secret content (defaults to `generic`). Also supports `basicAuthentication`, `awsIRSA`, `azureWorkloadIdentity` for platform engineer use cases |
| `data` | map of secret objects | Yes | Map of secret names to objects with `value` (required, sensitive) and `encoding` (optional, `string` or `base64`) |

### Output Properties (readOnly, set by recipe)

No output properties — secrets are consumed directly.

## Radius.Storage/blobStorages Schema

### Input Properties

No input properties beyond `environment` and `application`.

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `endpoint` | string | Storage account blob endpoint URL |
| `key` | string | Storage account access key |

## Radius.AI/agents Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `prompt` | string | Yes | System prompt that defines the agent's behavior and personality |
| `model` | string | Yes | Azure OpenAI model deployment name (e.g., `gpt-4.1-mini`) |
| `enableObservability` | boolean | No | Enable Application Insights telemetry (defaults to `false`) |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `agentEndpoint` | string | HTTP endpoint of the deployed agent runtime service |

## Checking Registered Types

```bash
# List all registered resource types
rad resource-type list

# Show details for a specific type
rad resource-type show Radius.Data/postgreSqlDatabases

# List recipes for a type
rad recipe list --environment default
```
