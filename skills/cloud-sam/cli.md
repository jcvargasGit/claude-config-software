# SAM CLI Commands

## Build and Deploy
```bash
# Build the application
sam build

# Build with specific template
sam build --template-file template.yaml

# Build for container-based deployment
sam build --use-container

# Deploy with guided prompts (first time)
sam deploy --guided

# Deploy with saved config
sam deploy --config-file samconfig.toml

# Deploy to specific environment
sam deploy --config-env prod
```

## Local Development
```bash
# Start local API
sam local start-api --port 3000

# Invoke function locally
sam local invoke GetUserFunction --event events/get-user.json

# Generate sample event
sam local generate-event apigateway http-api-proxy > events/api-event.json

# Start Lambda locally for debugging
sam local start-lambda

# Run with environment variables
sam local invoke --env-vars env.json
```

## Logs and Debugging
```bash
# Tail function logs
sam logs --name GetUserFunction --stack-name my-stack --tail

# Fetch logs for time range
sam logs --name GetUserFunction --start-time '5min ago'

# Trace requests
sam traces --stack-name my-stack
```

## Sync for Development
```bash
# Start sync for rapid iteration
sam sync --stack-name my-stack --watch

# Sync code only (skip infra changes)
sam sync --code --stack-name my-stack
```

## samconfig.toml

```toml
version = 0.1

[default]
[default.global.parameters]
stack_name = "my-app"

[default.build.parameters]
cached = true
parallel = true

[default.deploy.parameters]
capabilities = "CAPABILITY_IAM CAPABILITY_AUTO_EXPAND"
confirm_changeset = true
resolve_s3 = true

[default.sync.parameters]
watch = true

[dev]
[dev.deploy.parameters]
stack_name = "my-app-dev"
parameter_overrides = "Environment=dev"
confirm_changeset = false

[prod]
[prod.deploy.parameters]
stack_name = "my-app-prod"
parameter_overrides = "Environment=prod"
confirm_changeset = true
```
