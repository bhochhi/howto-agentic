# URL Shortener — Agent Instructions

## Project Overview
A serverless URL shortener deployed on AWS (API Gateway → Lambda → DynamoDB), written in Go 1.22.

## Spec-Driven Development
This project uses **spec-driven development**. The source of truth lives in `specs/`:
- `specs/requirements.md` — Functional and non-functional requirements
- `specs/api-spec.yaml` — OpenAPI 3.0 contract
- `specs/architecture.md` — AWS architecture and design decisions
- `specs/data-model.md` — DynamoDB table design and access patterns

**Always read the relevant spec before generating or modifying code.**

## Tech Stack
- **Language**: Go 1.22+
- **Runtime**: AWS Lambda
- **API**: API Gateway HTTP API (v2)
- **Database**: DynamoDB (on-demand capacity)
- **IaC**: Terraform 1.7+
- **CI/CD**: GitHub Actions

## Commands
- Build: `go build ./cmd/api/...`
- Test: `go test -v -cover ./...`
- Race check: `go test -race ./...`
- Lint: `golangci-lint run`
- Terraform plan: `cd terraform && terraform plan`
- Terraform apply: `cd terraform && terraform apply`

## Conventions
- Follow standard Go project layout (cmd/, internal/)
- Use table-driven tests
- Use `log/slog` for structured logging
- Return errors, don't panic
- Terraform uses snake_case, prefixed with `url_shortener_`
- All DynamoDB access patterns are documented in `specs/data-model.md`
