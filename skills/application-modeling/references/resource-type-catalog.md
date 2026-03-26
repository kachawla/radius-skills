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

### Radius.Storage (radius-resource-types)

Must be registered. All are recipe-based.

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.Storage/blobStorages` | `2025-08-01-preview` | Portable blob storage resource |

### Radius.Security (radius-resource-types)

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.Security/secrets` | `2025-08-01-preview` | Secret stores |

### Radius.AI (radius-resource-types)

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.AI/agents` | `2025-08-01-preview` | LLM-powered agent runtime |

### Supported Custom And Legacy Patterns

These are still referenced by this skill set even if they are not currently present as checked-in resource folders in `radius-resource-types`.

| Type | API Version | Description |
|------|------------|-------------|
| `Radius.Data/redisCaches` | `2025-08-01-preview` | Redis caches used in examples and custom recipe patterns |

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
| `size` | `'S' \| 'M' \| 'L'` | Depends on recipe | Database size |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `host` | string | Database hostname |
| `port` | string | Database port |
| `database` | string | Database name |
| `username` | string | Admin username |
| `password` | string | Admin password |

## Radius.Data/redisCaches Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `size` | `'S' \| 'M' \| 'L'` | Depends on recipe | Cache size |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `host` | string | Redis hostname |
| `port` | string | Redis port |
| `password` | string | Redis password (if auth enabled) |

## Radius.Data/mySqlDatabases Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `database` | string | No | Database name |
| `username` | string | No | Admin username |
| `version` | `'5.7' \| '8.0' \| '8.4'` | No | Major MySQL version |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `password` | string | Generated admin password |
| `host` | string | Database hostname |
| `port` | integer | Database port |

## Radius.Storage/blobStorages Schema

Manifest summary: the `blobStorages` manifest describes a portable blob storage resource with a minimal developer-facing schema and connection outputs for containers.

### Bicep Usage

```bicep
extension radiusStorage

resource storage 'Radius.Storage/blobStorages@2025-08-01-preview' = {
	name: 'mystorage'
	properties: {
		environment: environment
		application: application
	}
}
```

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `environment` | string | Yes | Radius Environment ID |
| `application` | string | No | Optional Radius Application ID |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `endpoint` | string | Blob service endpoint URL |
| `key` | string | Access key for the storage account |

### Connection Environment Variables

When connected to an `Applications.Core/containers` resource, the manifest documents these environment variables:

| Variable | Description |
|----------|-------------|
| `CONNECTION_<NAME>_ENDPOINT` | Storage account blob endpoint URL |
| `CONNECTION_<NAME>_KEY` | Storage account access key |

## Radius.Compute/persistentVolumes Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `sizeInGib` | integer | Yes | Volume size in GiB |
| `allowedAccessModes` | `ReadWriteOnce` \| `ReadOnlyMany` \| `ReadWriteMany` | No | Allowed mount modes |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `claimName` | string | PersistentVolumeClaim name exposed to consuming recipes |

## Radius.Security/secrets Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `kind` | `generic` \| `certificate-pem` \| `certificate-pkcs12` \| `basicAuthentication` \| `awsIRSA` \| `azureWorkloadIdentity` | No | Secret payload kind |
| `data` | object | Yes | Secret values map |

## Radius.AI/agents Schema

### Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `prompt` | string | Yes | System prompt for the agent |
| `model` | string | Yes | Azure OpenAI model deployment name |
| `enableObservability` | boolean | No | Enable Application Insights and Log Analytics |

### Output Properties (readOnly, set by recipe)

| Property | Type | Description |
|----------|------|-------------|
| `agentEndpoint` | string | HTTP endpoint of the deployed agent runtime |

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

## Checking Registered Types

```bash
# List all registered resource types
rad resource-type list

# Show details for a specific type
rad resource-type show Radius.Data/postgreSqlDatabases

# List recipes for a type
rad recipe list --environment default
```
