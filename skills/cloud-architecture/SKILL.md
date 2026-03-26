---
name: cloud-architecture
description: Design optimal cloud architecture based on your platform engineering requirements. Use when planning new services, migrations, or infrastructure changes that must align with organizational standards.
license: MIT
metadata:
  author: radius-skills
  version: "1.0.0"
---

# Cloud Architecture

Design cloud architecture that aligns with the standards defined in your `Platform-Engineering-Constitution.md`.

## Workflow

1. **Read the platform constitution.** Load `Platform-Engineering-Constitution.md` from the repository root. Extract approved providers, regions, compute platforms, IaC tooling, naming conventions, tagging requirements, and network topology.
2. **Understand the workload.** Ask what the application does, its traffic patterns, data requirements, and non-functional requirements (availability, latency, compliance).
3. **Propose architecture** constrained by the constitution. Only recommend approved providers, regions, and tooling.
4. **Generate an Architecture Decision Record (ADR)** documenting the design, trade-offs, and alternatives considered.
5. **Scaffold IaC** using the approved tooling and module conventions from the constitution.
6. **Map to Radius resource types** where applicable — use portable types so the same app.bicep works across environments.

## Data Gathering Questions

Ask these interactively, one at a time:

### Workload Identity
- **What does the application do?** Brief description of the service.
- **What are its components?** (e.g., web frontend, API backend, worker, scheduled jobs)

### Dependencies
- **What data stores does it need?** (PostgreSQL, Redis, MongoDB, blob storage, etc.)
- **What external services does it integrate with?** (third-party APIs, message queues, email, etc.)
- **Does it need to communicate with other internal services?**

### Traffic & Scale
- **Expected traffic volume?** (requests/sec, concurrent users)
- **Does it need auto-scaling?** (horizontal pod autoscaler, node autoscaler)
- **Any burst traffic patterns?** (event-driven spikes)

### Non-Functional Requirements
- **Availability target?** (99.9%, 99.99%, etc.)
- **Latency requirements?** (p99 response time)
- **Data residency / compliance constraints?** (beyond what's in the constitution)

### Deployment Strategy
- **Environments needed?** (dev, staging, production — or the constitution's standard set)
- **Multi-region / multi-cloud?** Or single region per the constitution?
- **CI/CD pipeline?** (GitHub Actions, Azure DevOps, etc.)

## Architecture Output Format

### Architecture Diagram (ASCII)

Provide an ASCII architecture diagram showing the key components and their relationships:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Ingress   │────▶│  Frontend   │────▶│   Backend   │
│  (Gateway)  │     │  (Node.js)  │     │    (Go)     │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                    ┌──────────┼──────────┐
                                    ▼          ▼          ▼
                              ┌──────────┐ ┌───────┐ ┌────────┐
                              │PostgreSQL│ │ Redis │ │ Blob   │
                              │  (RDS/   │ │(Elast-│ │Storage │
                              │  Azure)  │ │iCache)│ │        │
                              └──────────┘ └───────┘ └────────┘
```

### Architecture Decision Record (ADR)

```markdown
# ADR-NNN: <Title>

## Status
Proposed | Accepted | Superseded

## Context
What is the problem or requirement?

## Decision
What architecture was chosen and why?

## Consequences
What are the trade-offs? What are we giving up?

## Alternatives Considered
What other approaches were evaluated?

## Constitution Alignment
How does this align with the platform engineering constitution?
- Providers: ✅ Uses approved provider(s)
- Regions: ✅ Deploys to approved region(s)
- Compute: ✅ Uses approved compute platform
- IaC: ✅ Uses approved tooling
- Naming: ✅ Follows naming convention
- Tags: ✅ Includes required tags
```

### Resource Mapping

Map application needs to Radius resource types and cloud services:

| Need | Radius Resource Type | Azure Implementation | AWS Implementation |
|------|---------------------|---------------------|--------------------|
| SQL Database | `Radius.Data/postgreSqlDatabases` | Azure Database for PostgreSQL | Amazon RDS for PostgreSQL |
| MySQL Database | `Radius.Data/mySqlDatabases` | Azure Database for MySQL | Amazon RDS for MySQL |
| Object Storage | `Radius.Storage/blobStorages` | Azure Storage Account Blob service | Amazon S3-compatible recipe or custom implementation |
| Secrets | `Radius.Security/secrets` | Key Vault-backed or Kubernetes Secret recipe | Secrets Manager-backed or Kubernetes Secret recipe |
| Containers | `Radius.Compute/containers` | AKS pods | EKS pods |
| Persistent Volume | `Radius.Compute/persistentVolumes` | Azure Disk or Azure Files-backed PVC | EBS or EFS-backed PVC |
| Ingress | `Radius.Compute/routes` | Azure App Gateway | AWS ALB |
| AI Agent Runtime | `Radius.AI/agents` | Azure OpenAI + AI Search + Kubernetes runtime | Custom recipe required |

Keep `Radius.Data/redisCaches` available as a custom or legacy cache pattern when the environment has a matching resource type and recipe registered.

## IaC Scaffolding

When generating Terraform scaffolds, follow the constitution's project structure:

```
infrastructure/
├── environments/
│   ├── dev/
│   │   ├── main.tf          # Environment-specific config
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── production/
├── modules/
│   └── <service>/           # Custom modules for the app
├── backend.tf               # Remote state config
└── providers.tf              # Provider versions
```

### Naming Convention Application

Apply the constitution's naming pattern to all resources:

```
<org>-<env>-<region>-<service>-<resource-type>
```

### Tagging Application

Apply all required tags from the constitution to every resource.

## Radius Integration

When the application uses Radius:

1. **Use portable resource types** in `app.bicep` — don't hardcode cloud-specific resources
2. **Let recipes handle cloud specifics** — the same `app.bicep` deploys to both Azure and AWS
3. **Map environments to recipe sets:**
   - `dev` → Kubernetes recipes (in-cluster PostgreSQL, Redis pods)
   - `staging` / `production` → Cloud recipes (Azure Database for PostgreSQL, ElastiCache, etc.)

```bicep
// Same app.bicep works in all environments
resource db 'Radius.Data/postgreSqlDatabases@2025-08-01-preview' = {
  name: 'database'
  properties: {
    environment: environment  // Environment determines which recipe runs
    application: application
    size: 'M'
  }
}
```

## Guardrails

- **Never recommend providers, regions, or tooling not approved in the constitution.** If the user needs something outside the constitution, flag it as a deviation requiring amendment.
- **Always apply naming conventions and tagging requirements** from the constitution to proposed resources.
- **Prefer Radius portable resource types** over cloud-specific resources for application dependencies.
- **Propose the simplest architecture** that meets requirements. Don't over-engineer.
- **Document trade-offs** in the ADR — cost, complexity, operational burden, compliance.
- **Separate dev from production** — dev can use cheaper/simpler implementations (Kubernetes recipes), production uses managed cloud services.
- **Consider failure modes** — what happens when a component goes down? Design for the stated availability target.
- **Validate network topology** against the constitution — ensure the proposed architecture fits within the VNet/VPC standards.
