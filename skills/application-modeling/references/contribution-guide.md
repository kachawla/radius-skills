# Contributing to radius-resource-types

## Overview

[radius-resource-types](https://github.com/kachawla/radius-resource-types) is the repository for Radius resource types and recipes used by this skill set. It follows consistent conventions for directory structure, naming, and testing.

## Contribution Workflow

1. **Check existing types** — browse the repo to see if your resource type already exists
2. **Create the resource type YAML** following the schema conventions
3. **Write recipes** for target platforms (at minimum, `kubernetes/bicep` for local dev)
4. **Add a README** documenting the resource type and available recipes
5. **Test locally** with `rad run`
6. **Submit a PR**

## Directory Structure

```
<resourceType>/
├── README.md
├── <resourceType>.yaml
└── recipes/
    ├── kubernetes/
    │   └── bicep/
    │       └── kubernetes-<type>.bicep
    ├── azure-<service>/
    │   ├── bicep/
    │   │   └── azure-<service>.bicep
    │   └── terraform/
    │       ├── main.tf
    │       └── var.tf
    └── aws-<service>/
        └── terraform/
            ├── main.tf
            └── var.tf
```

## README Template

```markdown
# <ResourceType>

<One-line description>

## Available Recipes

| Platform | IaC | Recipe Name | Stage |
|----------|-----|-------------|-------|
| Kubernetes | Bicep | kubernetes-<type> | GA |
| Azure | Bicep | azure-<service> | Preview |
| AWS | Terraform | aws-<service> | Preview |

## Input Properties

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| size | S/M/L | No | S | Instance size |

## Output Properties

| Property | Type | Description |
|----------|------|-------------|
| host | string | Connection hostname |
| port | string | Connection port |
```

## Checklist

- [ ] Resource type YAML follows schema conventions
- [ ] `environment` is required in schema
- [ ] Input properties are cloud-agnostic
- [ ] Output properties are marked `readOnly: true`
- [ ] At least one recipe (kubernetes/bicep recommended)
- [ ] Recipes handle missing optional properties with defaults
- [ ] Recipes use `context.resource.properties.*` (not `context.properties.*`)
- [ ] Recipes output all `readOnly` properties in `result.properties`
- [ ] README documents properties and available recipes
- [ ] Tested locally with `rad run`
