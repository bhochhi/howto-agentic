# Project Instructions

## Spec-Driven Project
This project uses spec-driven development. Read `specs/` before generating code.

## Tech Stack
- Language: Go 1.22+
- Runtime: AWS Lambda
- API: API Gateway HTTP API (v2)
- Database: DynamoDB (on-demand)
- IaC: Terraform 1.7+

## Spec Files (Source of Truth)
- `specs/requirements.md` — Functional and non-functional requirements
- `specs/api-spec.yaml` — OpenAPI 3.0 contract
- `specs/architecture.md` — AWS architecture and design decisions
- `specs/data-model.md` — DynamoDB table design and access patterns

## Coding Standards
- Follow Go standard project layout (cmd/, internal/)
- Use table-driven tests
- All exported functions must have doc comments
- Use structured logging with `log/slog`
- Return errors with context: `fmt.Errorf("doing thing: %w", err)`

## Workflows
- Test: see `docs/workflows/test.md`
- Deploy: see `docs/workflows/deploy.md`
