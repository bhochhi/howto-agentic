# URL Shortener — Claude Code Instructions

## Spec-Driven Project
This project uses spec-driven development. **Always read specs/ before writing code.**

## Quick Reference
- Specs: `specs/requirements.md`, `specs/api-spec.yaml`, `specs/architecture.md`, `specs/data-model.md`
- Build: `go build ./cmd/api/...`
- Test: `go test -v -cover ./...`
- Deploy: `cd terraform && terraform apply`

## Rules
1. Read the relevant spec before generating any code
2. All API responses must match `specs/api-spec.yaml` exactly
3. All DynamoDB operations must use access patterns from `specs/data-model.md`
4. Write table-driven tests for every handler
5. Use `log/slog` for logging, never `fmt.Println`
6. Return errors with context: `fmt.Errorf("doing thing: %w", err)`
