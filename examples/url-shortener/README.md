# URL Shortener

A serverless URL shortener built on AWS using Go and Terraform — every file in this project was generated through AI agent conversations using spec-driven development.

## Spec-Driven Development

The `specs/` directory is the **source of truth**. Code is a derivative artifact.

```
specs/
├── requirements.md      → What we're building and why
├── api-spec.yaml        → OpenAPI contract (the API is defined here)
├── architecture.md      → System design and AWS resources
└── data-model.md        → DynamoDB table design & access patterns
```

## Architecture

```
API Gateway (acme.co)
    ├── POST /shorten      → Lambda: CreateURL   → DynamoDB: urls
    ├── GET /{code}        → Lambda: ResolveURL  → DynamoDB: urls + clicks
    └── GET /{code}/stats  → Lambda: GetStats    → DynamoDB: clicks
```

## Development

```bash
# Build
go build ./cmd/api/...

# Test
go test -v -cover ./...

# Deploy
cd terraform && terraform apply
```

## Agent Workflows

```bash
# Run tests (auto-approved via turbo-all)
/test

# Build + deploy
/deploy
```

## How This Project Was Built

See the full walkthrough: [Example Project Documentation](../../docs/05-example-project.md)
