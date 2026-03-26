# Radius Skills

Infrastructure and platform engineering skills for AI coding assistants, with [Radius](https://radapp.io/) integration. Each skill lives in its own sub-directory under `skills/` and helps developers and platform engineers work smarter with cloud infrastructure.

## Installation

### Skills

```bash
npx skills add radius-skills
```

See [skills.sh](https://skills.sh) for more info.

### MCP Server

The Radius MCP server gives your AI assistant live access to your Radius environments, recipes, and resource types. See [mcp-server/README.md](mcp-server/README.md) for setup instructions.

### Cursor Plugin

In Cursor chat, add this repository as a plugin:

```bash
/add-plugin radius-skills
```

## Available Skills

### Platform Engineering

#### platform-constitution

Define your organization's platform engineering standards by generating a `Platform-Engineering-Constitution.md`. Walks through an interactive workflow to capture cloud providers, compute platforms, IaC tooling, approved modules, and infrastructure policies.

#### app-audit *(coming soon)*

Audit applications against your platform engineering constitution. Validate that code, configurations, and deployments align with organizational standards.

#### cloud-architecture *(coming soon)*

Design optimal cloud architecture based on your platform engineering requirements. Generate architecture recommendations aligned with your constitution.

### Radius

#### radius-app

Author Radius application definitions using Bicep. Supports `Create an application definition` flows by discovering shared resources, scaffolding `app.bicep`, prompting for missing OCI and AI values, and wiring portable resource types correctly.

#### radius-environment

Configure Radius environments with recipes and resource types. Helps platform engineers register recipes, select recipe packs, and align environment configuration with the platform constitution.

#### radius-new-type

Create and install new Radius resource types and recipes when a needed type doesn't exist in [radius-resource-types](https://github.com/kachawla/radius-resource-types). Walks through defining the YAML schema, writing recipes, registering the type, and contributing changes there.

## MCP Server

The `mcp-server/` directory contains an MCP server that wraps the `rad` CLI, providing these tools:

| Tool | Description |
|------|-------------|
| `rad_environment_list` | List all Radius environments |
| `rad_environment_show` | Show details of a specific environment |
| `rad_recipe_list` | List recipes registered in an environment |
| `rad_recipe_show` | Show details of a specific recipe |
| `rad_resource_type_list` | List all registered resource types |
| `rad_resource_type_show` | Show resource type details and schema |
| `rad_app_list` | List deployed Radius applications |
| `rad_app_graph` | Show the resource graph of an application |
| `rad_resource_list` | List deployed resources of a given type |
| `rad_resource_show` | Show details of a specific deployed resource |

## Repo Structure

```
radius-skills/
├── .cursor-plugin/
│   └── plugin.json
├── .github/
│   └── copilot-instructions.md
├── mcp-server/
│   ├── package.json
│   ├── README.md
│   └── src/
│       ├── index.js
│       └── rad.js
├── README.md
├── LICENSE
└── skills/
    ├── platform-constitution/
    │   ├── SKILL.md
    │   └── references/
    ├── radius-app/
    │   ├── SKILL.md
    │   └── references/
    ├── radius-environment/
    │   ├── SKILL.md
    │   └── references/
    ├── radius-new-type/
    │   ├── SKILL.md
    │   └── references/
    ├── app-audit/
    │   ├── SKILL.md
    │   └── references/
    └── cloud-architecture/
        ├── SKILL.md
        └── references/
```

Each skill is a sub-directory under `skills/` containing:

- **SKILL.md** — Required. Instructions, triggers, and workflow for the skill.
- **references/** — Optional. Supporting documents loaded into context as needed.

## Contributing

1. Fork this repo
2. Create a new skill directory under `skills/`
3. Add a `SKILL.md` and optional `references/`
4. Submit a PR
