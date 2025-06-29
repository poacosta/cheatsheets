# Heroku Deployment & Environment Management

## Quick Commands

### Environment Variable Sync

```bash
# Direct export to .env
heroku config --shell --app your-app-name > .env

# Environment-specific sync
heroku config --shell --app your-app-production > .env.production
heroku config --shell --app your-app-staging > .env.staging
heroku config --shell --app your-app-development > .env.development
```

### Selective Variable Management

```bash
# Filter specific variables
heroku config:get DATABASE_URL REDIS_URL API_KEY --shell --app your-app-name > .env

# Exclude sensitive variables
heroku config --shell --app your-app-name | grep -v "SECRET\|PRIVATE\|KEY" > .env
```

## Common Patterns

### Automated Sync Script

```bash
#!/bin/bash
# sync-env.sh
APP_NAME=${1:-your-default-app}
ENV_FILE=${2:-.env}

echo "Syncing config from $APP_NAME to $ENV_FILE"
heroku config --shell --app $APP_NAME > $ENV_FILE

# Add local development variables
echo "" >> $ENV_FILE
echo "# Local development variables" >> $ENV_FILE
echo "DEBUG=true" >> $ENV_FILE
echo "LOG_LEVEL=debug" >> $ENV_FILE
```

### Multi-Environment Setup

```bash
# Create environment-specific files
heroku config --shell --app myapp-production > .env.production
heroku config --shell --app myapp-staging > .env.staging
heroku config --shell --app myapp-development > .env.development

# Symlink active environment
ln -sf .env.production .env
```

### Pipeline Integration

```bash
# Get config from specific pipeline stage
heroku config --shell --app $(heroku pipelines:info my-pipeline --json | \
  jq -r '.apps.production[0].name') > .env
```

## Personal Gotchas

### Security Considerations

```bash
# .gitignore essentials
.env
.env.*
!.env.example

# Generate example template
heroku config --app your-app-name | awk -F: '{print $1"="}' > .env.example
```

### Environment Parity

The `heroku local` command requires `.env` sync for development-production parity. Missing variables will cause runtime differences.

## Performance Notes

- Use shell format (`--shell`) for direct `.env` compatibility
- Batch variable retrieval is more efficient than individual `config:get`
- Pipeline queries reduce API calls for multi-stage deployments

## Architecture Considerations

### Environment Strategy

- **Production**: Live customer-facing environment
- **Staging**: Pre-production testing with production-like data
- **Development**: Local development with safe test data

### Variable Hierarchy

1. Pipeline-level config (shared)
2. App-specific overrides
3. Local development additions

## Deployment Patterns

### Blue-Green Deployment

```bash
# Switch traffic between apps
heroku pipelines:promote --app staging-app --to production-app
```

### Config Drift Prevention

```bash
# Regular config audit
heroku config --app prod-app > prod-config.txt
heroku config --app staging-app > staging-config.txt
diff prod-config.txt staging-config.txt
```

## Context Links

- [Heroku CLI Documentation](https://devcenter.heroku.com/articles/heroku-cli)
- [Config Vars](https://devcenter.heroku.com/articles/config-vars)
- [Heroku Pipelines](https://devcenter.heroku.com/articles/pipelines)
