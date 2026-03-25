# Recipe Structure

Recipes in [radius-resource-types](https://github.com/kachawla/radius-resource-types) follow a consistent directory layout:

```
<resourceType>/
├── README.md                          # Overview, recipe table, properties
├── <resourceType>.yaml                # Resource type YAML definition
└── recipes/
    ├── azure-<service>/
    │   ├── bicep/
    │   │   └── azure-<service>.bicep
    │   └── terraform/
    │       ├── main.tf
    │       └── var.tf
    ├── aws-<service>/
    │   └── terraform/
    │       ├── main.tf
    │       └── var.tf
    └── kubernetes/
        ├── bicep/
        │   └── kubernetes-<type>.bicep
        └── terraform/
            ├── main.tf
            └── var.tf
```

## Naming Conventions

- **Resource type YAML:** `<typeName>.yaml` (e.g., `postgreSqlDatabases.yaml`)
- **Recipe directories:** `<platform>-<service>/` or `kubernetes/` (e.g., `azure-postgresql/`, `aws-memorydb/`)
- **Recipe files (Bicep):** `<platform>-<type>.bicep` (e.g., `kubernetes-postgresql.bicep`, `azure-postgresql.bicep`)
- **Recipe files (Terraform):** `main.tf` + `var.tf`

## Recipe Registration

Recipes are published to OCI registries and registered with Radius:

```bash
# Publish
rad bicep publish --file recipes/kubernetes/bicep/kubernetes-postgresql.bicep \
  --target br:myregistry.azurecr.io/recipes/postgresql-kubernetes:v1.0.0

# Register
rad recipe register postgresql \
  --resource-type Radius.Data/postgreSqlDatabases \
  --template-kind bicep \
  --template-path "myregistry.azurecr.io/recipes/postgresql-kubernetes:v1.0.0"
```

## Recipe Versioning

- Use semantic versioning tags on OCI artifacts: `v1.0.0`, `v1.1.0`
- Use `latest` only for development/testing
- Pin production environments to specific versions
