# App Definition Flow

Use this workflow when the user asks Copilot to create an application definition from the current repository and Radius environment.

## Trigger Phrases

- `Create an application definition`
- `Create app.bicep`
- `Scaffold a Radius application`
- `Generate an app.bicep for this repo`

## What Copilot Should Do

1. Inspect the workspace for an environment definition, Radius environment config, or other files that indicate shared resources already exist.
2. Inspect the repository for a deployable container workload such as a `Dockerfile`, `Containerfile`, or a well-defined application service folder.
3. Inspect the application code for resource expectations such as PostgreSQL, blob storage, or AI agent endpoint usage.
4. Create `app.bicep` with the correct Radius extensions and resource declarations.
5. Ask the user only for values that cannot be inferred safely.

Treat `env.bicep`, `environment.bicep`, `shared-resources.bicep`, and similar Bicep files in the workspace as authoritative inputs for environment names and shared resource declarations.

## Question Order

Ask these in order when they are not already discoverable:

1. `What OCI registry host should be used for the container image?`
2. `Do you want to enable observability for the AI agent?`
3. `What prompt should the AI agent use?`

If the prompt value is long or multi-line, model it as a parameter instead of embedding it directly in the resource body.

## Generation Rules

- Add the necessary entries to `bicepconfig.json` for every Radius namespace used.
- Create an application resource.
- Declare shared PostgreSQL and blob storage resources as `existing` when files like `env.bicep` or `shared-resources.bicep` indicate they are shared environment resources.
- Prefer `Applications.Core/containers` for straightforward frontend or service containers.
- Use `Radius.Compute/containers` only when the repository clearly needs recipe-based container behavior.
- Add a `Radius.AI/agents` resource when the code expects an agent connection.
- Connect the container to all required resources.
- Use parameters for registry details and long or multi-line AI prompt text.

## Canonical Example

```bicep
extension radius
extension radiusData
extension radiusStorage
extension radiusAi

@description('The Radius environment ID.')
param environment string

@description('Application name.')
param applicationName string = 'myapp'

@description('OCI registry host, for example ghcr.io/myorg or myregistry.azurecr.io.')
param registryHost string

@description('Container image repository name.')
param imageRepository string = 'myapp'

@description('Container image tag.')
param imageTag string = 'latest'

@description('System prompt for the AI agent. Keep long or multi-line values out of the resource body.')
param agentPrompt string

resource app 'Applications.Core/applications@2023-10-01-preview' = {
  name: applicationName
  properties: {
    environment: environment
  }
}

resource postgres 'Radius.Data/postgreSqlDatabases@2025-08-01-preview' existing = {
  name: 'shared-postgres'
}

resource documents 'Radius.Storage/blobStorages@2025-08-01-preview' existing = {
  name: 'shared-documents'
}

resource agent 'Radius.AI/agents@2025-08-01-preview' = {
  name: 'assistant'
  properties: {
    environment: environment
    application: app.id
    prompt: agentPrompt
    model: 'gpt-4.1-mini'
    enableObservability: true
    connections: {
      postgres: {
        source: postgres.id
      }
      blobstorage: {
        source: documents.id
      }
    }
  }
}

resource container 'Applications.Core/containers@2023-10-01-preview' = {
  name: 'frontend-ui'
  location: 'global'
  properties: {
    application: app.id
    environment: environment
    container: {
      image: '${registryHost}/${imageRepository}:${imageTag}'
      ports: {
        http: {
          containerPort: 3000
        }
      }
    }
    connections: {
      agent: {
        source: agent.id
      }
    }
  }
}
```

## Notes

- Replace the `existing` resource names with the names found in the environment definition.
- If observability is disabled, write `enableObservability: false` after confirming with the user.
- If the repository already pins an image repository name, keep it and ask only for the registry host.
- If no AI usage is detected in the code, do not add the AI agent resource.

## Customer-Agent Rules

For repositories shaped like `Reshrahim/customer-agent`:

- Read `radius/env.bicep` to infer the environment name and available recipes.
- Read `radius/shared-resources.bicep` to infer the shared PostgreSQL and blob storage resource names to reference with `existing`.
- Detect `src/web/Dockerfile` as the frontend workload that should become an `Applications.Core/containers` resource.
- Detect `src/agent-runtime/app.py` as evidence that a `Radius.AI/agents` resource is required.
- Infer agent dependencies from environment variables such as `CONNECTION_POSTGRES_*`, `CONNECTION_STORAGE_*`, `CONNECTION_MODEL_*`, and `CONNECTION_SEARCH_*`.
- Do not create a separate top-level application container for the agent runtime when the AI agent recipe already deploys that runtime internally.