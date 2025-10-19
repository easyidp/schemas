# EasyIDP Schemas

**Single source of truth for EasyIDP platform configuration schemas.**

This repository contains the canonical schemas for all EasyIDP GitOps objects, enabling:
- **Validation** of platform-config YAML files
- **Dynamic form generation** in the IDP frontend
- **Type safety** across the EasyIDP ecosystem
- **Extension** by the community

## 📦 Schemas

### Current Version: `v1`

- **[integration.yaml](schemas/v1/integration.yaml)** - Integration configurations (GitHub, Datadog, AWS, etc.)
- **[service.yaml](schemas/v1/service.yaml)** - Service catalog entries
- **[scorecard.yaml](schemas/v1/scorecard.yaml)** - Service quality scorecards
- **[team.yaml](schemas/v1/team.yaml)** - Team definitions

## 🚀 Usage

### Validate Your Config

```bash
# Install validation tool (coming soon)
npm install -g @easyidp/schema-validator

# Validate your config files
easyidp-validate ./integrations/*.yaml --schema integration
easyidp-validate ./services/*.yaml --schema service
```

### In Your Platform-Config

Reference the schema version in your YAML files:

```yaml
apiVersion: idp.buckingham.io/v1
kind: Integration
# Schema: https://github.com/easyidp/schemas/blob/main/schemas/v1/integration.yaml
metadata:
  id: github-prod
  schema_version: v1
spec:
  provider: github
  config:
    organization: your-org
  credentials:
    token: <encrypted>
```

## 📚 Examples

See the [examples/](examples/) directory for complete working examples:

- [GitHub Integration](examples/integrations/github.yaml)
- [Datadog Integration](examples/integrations/datadog.yaml)
- [Microservice Definition](examples/services/microservice.yaml)
- [Production Ready Scorecard](examples/scorecards/production-ready.yaml)

## 🏗️ Schema Structure

Each schema defines:

1. **Base properties** - Common fields for all variants
2. **Conditionals** - Fields that appear based on `type` or `provider`
3. **Validation rules** - Required fields, enums, formats
4. **UI hints** - Labels, placeholders, widget types for form generation

### Example Schema Pattern

```yaml
apiVersion: idp.buckingham.io/v1
kind: Schema
metadata:
  name: Integration
  version: v1
spec:
  properties:
    provider:
      type: string
      enum: [github, datadog, aws]
  
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

## 🤝 Contributing

Want to add support for a new integration type? 

1. Fork this repo
2. Add your conditional block to `schemas/v1/integration.yaml`
3. Add an example to `examples/integrations/`
4. Submit a PR!

## 📖 Documentation

- [Integration Schema Reference](docs/integration-schema.md)
- [Service Schema Reference](docs/service-schema.md)
- [Extending Schemas](docs/extending-schemas.md)

## 📜 License

MIT License - see [LICENSE](LICENSE)

## 🔗 Related Projects

- [easyidp/core-api](https://github.com/easyidp/core-api) - Backend API
- [easyidp/frontend](https://github.com/easyidp/frontend) - Web UI
- [easyidp/sync](https://github.com/easyidp/sync) - GitOps sync worker
