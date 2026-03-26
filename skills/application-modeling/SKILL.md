---
name: application-modeling
description: Model cloud-native applications with Radius using Bicep. Use when asked to create an application definition, scaffold app.bicep, configure environments, or create custom resource types.
license: MIT
metadata:
  author: radius-skills
  version: "3.0.0"
---

# Radius Application Modeling

Use this skill for all Radius-related tasks: authoring application Bicep, configuring environments with recipes, and creating custom resource types. This skill covers the full lifecycle from defining an app to deploying it on Kubernetes.

## Workflow

1. **Read the platform constitution.** Check for `Platform-Engineering-Constitution.md` in the repository root. Note approved cloud providers, compute platforms, and IaC tooling.
2. **Check current Radius state.** Run `rad workspace show`, `rad environment list`, `rad recipe list`, and `rad resource-type list`.
3. **If asked to create an application definition, inspect the workspace first.** Look for environment definitions, existing shared resources, container artifacts, and application code that implies required connections.
4. **Prompt only for missing values.** Ask for OCI registry details, boolean choices without a safe default, and long free-text inputs such as AI prompts.
5. **Author or update** the application Bicep, resource types, and/or environment configuration as needed.
6. **Validate** against the constitution and test with `rad run`.

## Interactive App Definition Flow

Use this flow when the user says things like `Create an application definition`, `Create app.bicep`, or `Scaffold a Radius application`.

### Discovery Order

1. **Inspect the environment definition first.** If the workspace contains an environment definition or Radius environment config that already provides shared resources such as PostgreSQL or blob storage, model those resources as `existing` in `app.bicep` rather than creating new ones.
2. **Inspect the repository for container artifacts.** Look for `Dockerfile`, `Containerfile`, OCI build configs, or application folders that clearly map to a container workload. If a container exists in the repository, add a `Radius.Compute/containers` resource.
3. **Inspect the code for required connections.** Search for connection names, SDK usage, or environment variables that imply dependencies such as PostgreSQL, blob storage, or AI agent endpoints.
4. **Inspect for AI-agent expectations.** If the code expects an agent endpoint or agent-style integration, add a `Radius.AI/agents` resource and connect the container to it.

### Required Questions

Ask the user only when the value cannot be inferred safely:

1. **OCI registry host** for the container image.
2. **Enable observability?** Ask explicitly for `Radius.AI/agents.enableObservability` because it is boolean and should not be guessed.
3. **Prompt value** for the AI agent.

### Authoring Rules For This Flow

- **Always create `app.bicep`** when it does not already exist.
- **Always ensure `bicepconfig.json` includes the needed Radius extensions** for every resource type used in the generated app.
- **Use `existing` resources** for shared infrastructure already defined by the environment, especially PostgreSQL and blob storage when those are pre-provisioned.
- **Add a container resource** when the repository clearly contains a deployable application container.
- **Prompt for the OCI registry** instead of hardcoding one.
- **Prompt for AI observability** instead of assuming `true` or `false`.
- **If the AI prompt is long or multi-line, do not inline it in the resource.** Model it as `param agentPrompt string` and set `prompt: agentPrompt`.
- **If the AI prompt is short and single-line, it may be inlined** unless the user prefers parameterization.
- **Prefer minimal app scaffolds** that connect to existing shared resources rather than duplicating them.

### Expected Output Shape

The generated `app.bicep` should usually include:

- An application resource.
- Existing shared resources such as PostgreSQL and blob storage, when discovered from the environment definition.
- A new `Radius.Compute/containers` resource for the application container.
- A new `Radius.AI/agents` resource when the code expects an AI agent.
- Connections from the container to PostgreSQL, blob storage, and the AI agent.
- Parameters for registry details and long free-text values.

See [references/app-definition-flow.md](references/app-definition-flow.md) for the canonical example and decision rules.

---

## Part 1: Application Authoring

### Bicep Extension Setup

Before writing `app.bicep`, configure `bicepconfig.json`:

```json
{
  "extensions": {
    "radius": "br:biceptypes.azurecr.io/radius:latest",
    "aws": "br:biceptypes.azurecr.io/aws:latest"
  },
  "experimentalFeaturesEnabled": {
    "extensibility": true,
    "dynamicTypeLoading": true
  }
}
```

**If using `Radius.*` resource types** (from `radius-resource-types`), add custom extensions:

```json
{
  "extensions": {
    "radius": "br:biceptypes.azurecr.io/radius:latest",
    "aws": "br:biceptypes.azurecr.io/aws:latest",
    "radiusCompute": "radius-compute.tgz",
    "radiusData": "radius-data.tgz",
    "radiusStorage": "radius-storage.tgz",
    "radiusSecurity": "radius-security.tgz",
    "radiusAi": "radius-ai.tgz"
  },
  "experimentalFeaturesEnabled": {
    "extensibility": true,
    "dynamicTypeLoading": true
  }
}
```

Generate custom extensions with:

```bash
rad bicep publish-extension --from-file <manifest.yaml> --target <output.tgz>
```

### Resource Type Namespaces

There are two families of resource types:

#### `Applications.*` (Built-in)

Built into Radius. `Applications.Core/containers` is handled directly by the Radius control plane — not recipe-based.

| Type | Description |
|------|-------------|
| `Applications.Core/applications` | Application grouping |
| `Applications.Core/containers` | Container workloads (directly managed) |
| `Applications.Core/gateways` | HTTP ingress gateways |
| `Applications.Datastores/redisCaches` | Redis (recipe-based) |
| `Applications.Datastores/sqlDatabases` | SQL databases (recipe-based) |

#### `Radius.*` (from radius-resource-types)

Community/extensible types. **ALL are recipe-based**, including `Radius.Compute/containers`.

| Type | Description |
|------|-------------|
| `Radius.Compute/containers` | Container workloads (**recipe-based**) |
| `Radius.Compute/persistentVolumes` | Persistent storage volumes |
| `Radius.Compute/routes` | HTTP routing (requires Gateway API controller) |
| `Radius.Data/mySqlDatabases` | MySQL databases |
| `Radius.Data/postgreSqlDatabases` | PostgreSQL databases |
| `Radius.Storage/blobStorages` | Blob/object storage |
| `Radius.Security/secrets` | Secret stores |
| `Radius.AI/agents` | LLM-powered agent runtimes |

`Radius.Data/redisCaches` is still kept in these skills as a supported custom pattern and example, even though it is not currently listed in the checked-in folders of `radius-resource-types`.

> **Critical difference:** `Applications.Core/containers` is directly managed by Radius. `Radius.Compute/containers` is recipe-based — it needs a registered recipe to deploy.

### Application Structure

#### Using `Applications.*` Types (Simpler)

```bicep
extension radius

param environment string
param application string

resource frontend 'Applications.Core/containers@2023-10-01-preview' = {
  name: 'frontend'
  properties: {
    application: application
    container: {                    // singular "container"
      image: 'myregistry/frontend:latest'
      ports: {
        web: { containerPort: 3000 }
      }
    }
    connections: {
      database: { source: db.id }
    }
  }
}

resource db 'Applications.Datastores/sqlDatabases@2023-10-01-preview' = {
  name: 'database'
  properties: {
    environment: environment
    application: application
  }
}
```

#### Using `Radius.*` Types (Portable)

```bicep
extension radius
extension radiusCompute
extension radiusData

param environment string
param application string

resource frontend 'Radius.Compute/containers@2025-08-01-preview' = {
  name: 'frontend'
  properties: {
    environment: environment
    application: application
    containers: {                   // plural "containers" — a map!
      frontend: {
        image: 'myregistry/frontend:latest'
        ports: {
          web: { containerPort: 3000 }
        }
      }
    }
    connections: {
      database: { source: db.id }
    }
  }
}

resource db 'Radius.Data/postgreSqlDatabases@2025-08-01-preview' = {
  name: 'database'
  properties: {
    environment: environment
    application: application
    size: 'S'                       // Required if recipe expects it
  }
}
```

> **Schema difference:** `Applications.Core/containers` uses `container` (singular object). `Radius.Compute/containers` uses `containers` (plural map where each key is a container name).

### Connections and Environment Variables

#### `Applications.Core/containers` — Individual Env Vars

```
CONNECTION_<NAME>_HOST
CONNECTION_<NAME>_PORT
CONNECTION_<NAME>_DATABASE
CONNECTION_<NAME>_USERNAME
CONNECTION_<NAME>_PASSWORD
```

#### `Radius.Compute/containers` — JSON Properties Blob

```
CONNECTION_<NAME>_PROPERTIES={"host":"...","port":"...","database":"..."}
CONNECTION_<NAME>_ID=<resource-id>
CONNECTION_<NAME>_NAME=<connection-name>
CONNECTION_<NAME>_TYPE=<resource-type>
```

**Application code must parse `CONNECTION_<NAME>_PROPERTIES` as JSON.** Write a helper function that supports both formats:

```go
// Go
func getConnProp(connName, prop string) string {
    propsJSON := os.Getenv("CONNECTION_" + connName + "_PROPERTIES")
    if propsJSON != "" {
        var props map[string]interface{}
        if err := json.Unmarshal([]byte(propsJSON), &props); err == nil {
            if val, ok := props[strings.ToLower(prop)]; ok {
                return fmt.Sprintf("%v", val)
            }
        }
    }
    return os.Getenv("CONNECTION_" + connName + "_" + prop)
}
```

```javascript
// Node.js
function getConnProp(connName, prop) {
  const propsJson = process.env[`CONNECTION_${connName}_PROPERTIES`];
  if (propsJson) {
    try {
      const props = JSON.parse(propsJson);
      return props[prop.toLowerCase()] || '';
    } catch (e) {}
  }
  return process.env[`CONNECTION_${connName}_${prop}`] || '';
}
```

### Container Image Requirements

- **Cloud registry** (ACR, ECR, GHCR): Works if cluster has credentials configured
- **Local dev with kind**: Push to a local OCI registry and use `host.docker.internal:<port>` as the image host
- **`imagePullPolicy`**: The `Radius.Compute/containers` recipe may set `Always` — images must be pullable, not just loaded with `kind load`

### Health Endpoints

Always add `/healthz` (liveness) and `/readyz` (readiness) endpoints. The readiness probe should check downstream dependencies.

---

## Part 2: Environment & Recipe Setup

### Initialize Radius

```bash
rad initialize
rad workspace create kubernetes default --group default --environment default
rad environment create myenv --namespace my-namespace   # if needed
rad environment switch myenv
```

### Register Resource Types

```bash
# Download YAML from radius-resource-types, then register
rad resource-type create Radius.Data/postgreSqlDatabases --from-file postgreSqlDatabases.yaml
rad resource-type show Radius.Data/postgreSqlDatabases   # verify

# Repeat for other repo-backed types such as:
# Radius.Data/mySqlDatabases
# Radius.Storage/blobStorages
# Radius.Security/secrets
# Radius.Compute/persistentVolumes
# Radius.Compute/routes
# Radius.AI/agents
```

### Generate Bicep Extensions

```bash
rad bicep publish-extension --from-file postgreSqlDatabases.yaml --target radius-data.tgz
# Then add the matching extension key in bicepconfig.json, for example:
#   "radiusData": "radius-data.tgz"
#   "radiusStorage": "radius-storage.tgz"
#   "radiusSecurity": "radius-security.tgz"
#   "radiusAi": "radius-ai.tgz"
```

Combine multiple types into one YAML (with `---` separator) to generate a single extension.

### Publish and Register Recipes

> **Critical:** Recipes registered from local file paths (`/tmp/recipe.bicep`) will NOT work. The Radius control plane runs inside Kubernetes and cannot access the host filesystem. Always publish to an OCI registry.

```bash
# Publish to OCI registry
rad bicep publish --file kubernetes-postgresql.bicep \
  --target br:myregistry.azurecr.io/recipes/postgresql-kubernetes:latest

# For local/insecure registries, add --plain-http
rad bicep publish --file kubernetes-postgresql.bicep \
  --target br:localhost:5001/recipes/postgresql-kubernetes:latest --plain-http

# Register (use host.docker.internal for in-cluster access)
rad recipe register postgresql \
  --resource-type Radius.Data/postgreSqlDatabases \
  --template-kind bicep \
  --template-path "host.docker.internal:5001/recipes/postgresql-kubernetes:latest" \
  --plain-http --environment myenv
```

### Recipe Selection Guide

| Constitution Says | Recipe Platform | Recipe IaC | Example |
|-------------------|----------------|------------|---------|
| Azure + Terraform | `azure-*` | `terraform/` | `azure-cache/terraform` |
| Azure + Bicep | `azure-*` | `bicep/` | `azure-cache/bicep` |
| AWS + Terraform | `aws-*` | `terraform/` | `aws-memorydb/terraform` |
| Kubernetes (local) | `kubernetes` | `bicep/` or `terraform/` | `kubernetes-redis/bicep` |

### Local Development with kind

#### 1. Local OCI Registry

```bash
docker run -d -p 5001:5000 --name radius-registry registry:2
curl http://localhost:5001/v2/_catalog   # verify
```

#### 2. Host Networking

`localhost` inside a k8s pod does NOT reach the host machine. Use `host.docker.internal` for all registry and service URLs.

#### 3. Insecure Registry for containerd

```bash
NODENAME=$(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')

docker exec $NODENAME mkdir -p /etc/containerd/certs.d/host.docker.internal:5001
docker exec $NODENAME bash -c 'cat > /etc/containerd/certs.d/host.docker.internal:5001/hosts.toml << EOF
[host."http://host.docker.internal:5001"]
  capabilities = ["pull", "resolve"]
  skip_verify = true
EOF'

docker exec $NODENAME bash -c \
  'sed -i "s|config_path = \"\"|config_path = \"/etc/containerd/certs.d\"|" /etc/containerd/config.toml'
docker exec $NODENAME systemctl restart containerd
```

#### 4. Build and Push Images

```bash
docker build -t myapp-backend:latest ./backend
docker tag myapp-backend:latest localhost:5001/myapp-backend:latest
docker push localhost:5001/myapp-backend:latest
```

In `app.bicep`, reference as `host.docker.internal:5001/myapp-backend:latest`.

### Environment Management Commands

```bash
rad environment list
rad environment show myenv
rad recipe list --environment myenv
rad recipe show postgresql --resource-type Radius.Data/postgreSqlDatabases --environment myenv
rad resource-type list
rad resource-type show Radius.Data/postgreSqlDatabases
rad workspace show
```

---

## Part 3: Custom Resource Types & Recipes

### Resource Type YAML Schema

```yaml
namespace: Radius.Data
types:
  postgreSqlDatabases:
    description: |
      A portable PostgreSQL database resource.
    apiVersions:
      '2025-08-01-preview':
        schema:
          type: object
          properties:
            environment:
              type: string
              description: "(Required) The Radius Environment ID."
            application:
              type: string
              description: "(Optional) The Radius Application ID."
            size:
              type: string
              enum: ['S', 'M', 'L']
              description: "(Optional) The size of the database."
            host:
              type: string
              description: The hostname.
              readOnly: true
            port:
              type: string
              description: The port.
              readOnly: true
            database:
              type: string
              description: The database name.
              readOnly: true
            username:
              type: string
              description: The username.
              readOnly: true
            password:
              type: string
              description: The password.
              readOnly: true
          required: [environment]
```

**Conventions:**
- `environment` is always required
- Input properties are cloud-agnostic, minimal
- Output properties are marked `readOnly: true` — they become connection env vars
- Combine multiple types in one YAML under the same namespace

### Recipe Directory Structure

```
<resourceType>/
├── README.md
├── <resourceType>.yaml
└── recipes/
    ├── kubernetes/bicep/kubernetes-<type>.bicep
    ├── azure-<service>/bicep/azure-<service>.bicep
    └── aws-<service>/terraform/main.tf
```

### Recipe Context Object

```
context.resource.id            // Full resource ID
context.resource.name          // Resource name
context.resource.type          // e.g., "Radius.Data/postgreSqlDatabases"
context.resource.properties    // Developer-set properties from app.bicep
context.runtime.kubernetes.namespace  // Target namespace
```

> **Important:** Use `context.resource.properties.*` (not `context.properties.*`).

### Bicep Recipe Template

```bicep
param context object

var size = contains(context.resource.properties, 'size') ? context.resource.properties.size : 'S'
var name = context.resource.name
var namespace = context.runtime.kubernetes.namespace

// ... deploy k8s resources ...

output result object = {
  properties: {
    host: '${name}-svc.${namespace}.svc.cluster.local'
    port: '5432'
    database: name
    username: 'admin'
    password: 'generated-password'
  }
}
```

### Terraform Recipe Template

```hcl
variable "context" {
  type = any
}

locals {
  size      = try(var.context.resource.properties.size, "S")
  name      = var.context.resource.name
  namespace = var.context.runtime.kubernetes.namespace
}

output "result" {
  value = {
    properties = {
      host     = "${local.name}-svc.${local.namespace}.svc.cluster.local"
      port     = "5432"
      database = local.name
      username = "admin"
      password = "generated-password"
    }
  }
}
```

### Publishing and Registration

```bash
# 1. Publish recipe to OCI registry
rad bicep publish --file kubernetes-postgresql.bicep \
  --target br:myregistry.azurecr.io/recipes/postgresql-kubernetes:latest

# 2. Register resource type
rad resource-type create Radius.Data/postgreSqlDatabases --from-file manifest.yaml

# 3. Register recipe
rad recipe register postgresql \
  --resource-type Radius.Data/postgreSqlDatabases \
  --template-kind bicep \
  --template-path "myregistry.azurecr.io/recipes/postgresql-kubernetes:latest"

# 4. Generate Bicep extension
rad bicep publish-extension --from-file manifest.yaml --target radius-data.tgz

# 5. Test
rad run app.bicep
```

---

## Common Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| `extension "radius" is not recognized` | Missing `bicepconfig.json` | Create it with the Radius extension registry URL |
| `RecipeDownloadFailed: not a valid repository/tag` | Recipe registered from local file path | Publish to OCI registry, re-register |
| `RecipeDownloadFailed: connection refused` | `localhost` used inside k8s pod | Use `host.docker.internal` instead |
| `RecipeDownloadFailed: HTTPS required` | containerd defaults to HTTPS | Configure `certs.d/hosts.toml` in kind node |
| `property 'size' doesn't exist` | Recipe expects property not set in app.bicep | Add the property or add a safe default in recipe |
| `RecipeNotFoundFailure` | No recipe registered for resource type | `rad recipe register` |
| `ImagePullBackOff` | Image not accessible from cluster | Push to registry; `kind load` won't work with `Always` pull policy |
| `CONNECTION_X_HOST` is empty | Using `Radius.Compute/containers` (JSON format) | Parse `CONNECTION_X_PROPERTIES` JSON blob instead |
| Bicep validation errors on `Radius.*` types | No Bicep extension generated | `rad bicep publish-extension` |
| `container` vs `containers` schema error | Wrong property name for container type | `Applications.Core` = singular `container`; `Radius.Compute` = plural `containers` map |

## References

| Topic | Reference | Use for |
|-------|-----------|---------|
| Bicep Patterns | [references/bicep-patterns.md](references/bicep-patterns.md) | Multi-container apps, gateways, parameterization |
| App Definition Flow | [references/app-definition-flow.md](references/app-definition-flow.md) | Scaffolding `app.bicep` from environment and repo discovery |
| Connection Conventions | [references/connection-conventions.md](references/connection-conventions.md) | Env var formats, JSON parsing, portable code |
| Resource Type Catalog | [references/resource-type-catalog.md](references/resource-type-catalog.md) | Available types, schemas, properties |
| Local Development | [references/local-development.md](references/local-development.md) | kind, local registry, containerd, Dockerfiles |
| Resource Type YAML | [references/resource-type-yaml.md](references/resource-type-yaml.md) | YAML schema definition format |
| Recipe Authoring | [references/recipe-authoring.md](references/recipe-authoring.md) | Bicep/Terraform recipes, context object |
| Environment Config | [references/environment-config.md](references/environment-config.md) | Workspaces, environments, namespaces |
| Cloud Providers | [references/cloud-providers.md](references/cloud-providers.md) | Azure, AWS credentials for Radius |
| Recipe Structure | [references/recipe-structure.md](references/recipe-structure.md) | Directory layout in radius-resource-types |
| Contribution Guide | [references/contribution-guide.md](references/contribution-guide.md) | Contributing to radius-resource-types |

## Guardrails

- **Always check the platform constitution** before suggesting resource types, recipes, or cloud-specific patterns.
- **Use portable resource types** (`Radius.*`) instead of cloud-specific resources unless explicitly needed.
- **Never hardcode infrastructure details** in application definitions — let recipes handle it.
- **Always include `environment`** — required for all Radius resources.
- **Handle both connection env var formats** (`_PROPERTIES` JSON and individual vars) for portability.
- **Set all recipe-expected properties** in Bicep (e.g., `size`), or use safe defaults in recipes.
- **Always configure `bicepconfig.json`** — the Radius Bicep extension won't resolve without it.
- **When scaffolding `app.bicep`, inspect before asking.** Infer shared resources and container workloads from the workspace whenever possible.
- **When shared resources already exist in the environment, declare them with `existing`** instead of provisioning duplicates.
- **Always ask for the OCI registry host** when it cannot be inferred from the repository.
- **Always ask before setting boolean AI options** such as `enableObservability` when there is no clear project default.
- **Move long or multi-line prompt text into a parameter** instead of embedding it directly in the AI agent resource.
- **Never register recipes from local file paths** — publish to an OCI registry first.
- **Use `host.docker.internal`** instead of `localhost` for in-cluster access to host services.
- **Use `--plain-http`** when working with insecure (HTTP) registries.
- **Generate Bicep extensions** after registering resource types for IDE validation.
- **Keep resource type interfaces cloud-agnostic** — cloud details belong in recipes, not schemas.
- **Always handle missing optional properties** in recipes with safe defaults.
- **Use `containers` (plural map) for `Radius.Compute`** and `container` (singular) for `Applications.Core`.
- **Test with `rad run`** before deploying to production.
