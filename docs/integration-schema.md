# Integration Schema Reference

The Integration schema defines how external services connect to your EasyIDP platform.

## Schema Location

**File**: `schemas/v1/integration.yaml`  
**Version**: v1

## Overview

Integrations enable your IDP to connect with external services like:
- Source control (GitHub, GitLab)
- Monitoring (Datadog, New Relic)
- Cloud providers (AWS, GCP, Azure)
- Artifact registries (Docker Hub, ECR)
- Incident management (PagerDuty)
- Communication (Slack, Microsoft Teams)

## Base Properties

These fields are **required for all integrations**, regardless of provider:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier (lowercase, alphanumeric, hyphens) |
| `name` | string | Yes | Display name for the integration |
| `provider` | string | Yes | Provider type (github, datadog, aws, etc.) |
| `description` | string | No | Human-readable description |
| `category` | string | No | Category (auto-set by provider in most cases) |
| `enabled` | boolean | No | Whether the integration is active (default: true) |
| `capabilities` | array | No | List of capabilities this integration provides |

## Supported Providers

### GitHub (`provider: github`)

**Category**: `source_control`

**Config Fields**:
- `is_organization` (boolean) - Organization or personal account (default: true)
- `organization` (string, required) - GitHub org or username

**Credentials** (encrypted with SOPS):
- `token` (string, required) - Personal Access Token (ghp_...)

**Example**:
```yaml
spec:
  provider: github
  config:
    is_organization: true
    organization: easyidp
  credentials:
    token: ghp_xxxxxxxxxxxxx
```

### Datadog (`provider: datadog`)

**Category**: `monitoring`

**Config Fields**:
- `site` (string, required) - Datadog site (US1, US3, US5, EU1, AP1, US1_FED)

**Credentials** (encrypted with SOPS):
- `api_key` (string, required) - Datadog API key
- `app_key` (string, required) - Datadog application key

**Example**:
```yaml
spec:
  provider: datadog
  config:
    site: US1
  credentials:
    api_key: xxxxxxxxxxxxx
    app_key: xxxxxxxxxxxxx
```

### AWS (`provider: aws`)

**Category**: `cloud`

**Config Fields**:
- `region` (string, required) - Primary AWS region (e.g., us-east-1)
- `account_id` (string, required) - 12-digit AWS account ID

**Credentials** (encrypted with SOPS):
- `access_key_id` (string, required) - AWS access key
- `secret_access_key` (string, required) - AWS secret key

**Example**:
```yaml
spec:
  provider: aws
  config:
    region: us-east-1
    account_id: "123456789012"
  credentials:
    access_key_id: AKIAXXXXXXXXXXXXXXXX
    secret_access_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Docker Hub (`provider: dockerhub`)

**Category**: `artifact_registry`

**Config Fields**:
- `namespace` (string, required) - Docker Hub username or org

**Credentials** (encrypted with SOPS):
- `username` (string, required) - Docker Hub username
- `password` (string, required) - Password or access token

**Example**:
```yaml
spec:
  provider: dockerhub
  config:
    namespace: easyidp
  credentials:
    username: easyidp
    password: xxxxxxxxxxxxx
```

### PagerDuty (`provider: pagerduty`)

**Category**: `incident_management`

**Config Fields**:
- `subdomain` (string, required) - PagerDuty subdomain

**Credentials** (encrypted with SOPS):
- `api_token` (string, required) - PagerDuty API token

### Slack (`provider: slack`)

**Category**: `communication`

**Config Fields**:
- `workspace` (string, required) - Slack workspace name

**Credentials** (encrypted with SOPS):
- `bot_token` (string, required) - Bot user OAuth token (xoxb-...)

### Custom (`provider: custom`)

For custom integrations, you can provide arbitrary JSON configuration:

**Config Fields**:
- `config` (object) - Custom configuration object
- `credentials` (object) - Custom credentials (will be encrypted)

## Conditional Fields

The schema uses **if/then conditionals** to show different fields based on the selected `provider`. This enables:

1. **Type-specific validation** - Only validate fields relevant to the provider
2. **Dynamic UI** - Forms show only applicable fields
3. **Extensibility** - Easy to add new providers

### How Conditionals Work

```yaml
conditionals:
  - if:
      properties:
        provider:
          const: github
    then:
      properties:
        config:
          properties:
            organization:
              type: string
              required: true
```

**Result**: The `organization` field only appears and is validated when `provider: github` is selected.

## UI Hints

The schema includes UI metadata for form generation:

```yaml
ui:
  label: Organization Name        # Field label
  placeholder: your-org           # Input placeholder
  hint: GitHub organization name  # Help text
  widget: select                  # Widget type (input, select, textarea, password, etc.)
```

## Security

**Credentials are encrypted with SOPS** before being stored in Git. Fields marked with `secret: true` will:
- Render as password inputs
- Be encrypted before commit
- Be decrypted only by authorized services

## Adding a New Provider

To add support for a new integration type:

1. Add the provider to the `enum` list in `spec.properties.provider.enum`
2. Add a new conditional block with your provider's config fields
3. Add an example to `examples/integrations/`
4. Update this documentation

**Example**:
```yaml
- if:
    properties:
      provider:
        const: my-new-provider
  then:
    properties:
      config:
        properties:
          my_field:
            type: string
            required: true
```

## Validation

The schema provides validation for:
- **Required fields** - Must be present
- **Types** - String, boolean, number, array, object
- **Patterns** - Regex validation (e.g., account IDs)
- **Enums** - Fixed list of allowed values
- **Formats** - Special formats like email, URL, password

## Complete Example

```yaml
apiVersion: idp.buckingham.io/v1
kind: Integration
metadata:
  id: github-prod
  name: GitHub Production
  description: Main GitHub organization
  schema_version: v1
  labels:
    environment: production
spec:
  provider: github
  category: source_control
  enabled: true
  capabilities:
    - repository_discovery
    - webhook_integration
  config:
    is_organization: true
    organization: easyidp
  credentials:
    token: ENC[AES256_GCM,data:...,type:str]
```
