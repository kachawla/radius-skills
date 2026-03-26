# Copilot Instructions

## Platform Engineering Constitution

This project follows a platform engineering constitution defined in `Platform-Engineering-Constitution.md` at the repository root. This document codifies the organization's approved cloud providers, compute platforms, IaC tooling, deployment standards, and infrastructure policies.

**If `Platform-Engineering-Constitution.md` does not exist**, use the `platform-constitution` skill to generate one. Walk the user through the interactive workflow to gather their organization's requirements before proceeding with any infrastructure-related tasks.

**If `Platform-Engineering-Constitution.md` exists**, read it before answering any questions about infrastructure, cloud architecture, IaC, or deployment. All recommendations must align with the standards defined there.

## Infrastructure Guidelines

- All infrastructure changes must comply with the platform constitution.
- Use only approved IaC modules from the sources listed in the constitution.
- Deploy only to approved cloud regions and providers.
- Follow the naming conventions, tagging requirements, and network topology defined in the constitution.
- When proposing new cloud resources or architecture, validate against the constitution and flag any deviations.

## Radius

This project uses [Radius](https://radapp.io/) to bridge developers and platform engineers.

- **Developers**: Use portable resource types (e.g., `Radius.Data/postgreSqlDatabases`) in application Bicep files. Do not hardcode cloud-specific infrastructure — let Recipes handle the implementation.
- **Platform Engineers**: Register resource types and recipes in Radius environments that align with the platform constitution. Use `rad recipe list` and `rad resource-type list` to verify what's available.
- **Use the Radius MCP tools** (when available) to introspect the live environment: check registered recipes, resource types, deployed applications, and resource graphs before making changes.
- When authoring new resource types or recipes, follow the conventions in [radius-resource-types](https://github.com/kachawla/radius-resource-types).
