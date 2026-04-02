# Skills Catalog

> Reusable skill folders that teach agents your team's conventions for Go, Java, JavaScript, AWS, and Terraform.

---

## What Is a Skill?

A **skill** is a folder containing a `SKILL.md` file (and optionally helper scripts, examples, and resources) that gives an agent specialized knowledge for a particular technology or task.

```
.agent/skills/
├── golang/
│   └── SKILL.md
├── java/
│   └── SKILL.md
├── javascript/
│   └── SKILL.md
├── aws/
│   └── SKILL.md
└── terraform/
    └── SKILL.md
```

When an agent encounters a task that matches a skill's domain, it reads the `SKILL.md` and follows the instructions. Skills are how you encode **team conventions** so the agent produces code that looks like *your team* wrote it.

---

## Skill: Golang

### `.agent/skills/golang/SKILL.md`

```yaml
---
name: golang
description: Go project scaffolding, idiomatic patterns, testing, and deployment
---

## Project Layout

Follow the standard Go project layout:

```text
cmd/           → Application entry points (one main.go per binary)
internal/      → Private packages (cannot be imported externally)
pkg/           → Public library packages (importable by other projects)
scripts/       → Build and automation scripts
docs/          → Documentation
```

## Module Initialization

- Always use `go mod init` with the full module path: `github.com/org/project`
- Pin Go version in `go.mod` to the minimum supported version
- Run `go mod tidy` after adding dependencies

## Coding Standards

### Error Handling
- Return errors, never panic in library code
- Wrap errors with context: `fmt.Errorf("creating user: %w", err)`
- Use sentinel errors for well-known conditions: `var ErrNotFound = errors.New("not found")`

### Naming
- Use MixedCaps, not underscores
- Acronyms are all caps: `HTTPHandler`, `URLShortener`
- Interface names: single-method → verb+er (`Reader`, `Stringer`)

### Structs & Interfaces
- Accept interfaces, return structs
- Keep interfaces small (1-3 methods)
- Use constructor functions: `func NewService(deps) *Service`

### Logging
- Use `log/slog` (Go 1.21+) for structured logging
- Log at appropriate levels: Debug, Info, Warn, Error
- Include context fields: `slog.String("url_code", code)`

## Testing

### Table-Driven Tests
```go
func TestShorten(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    string
        wantErr bool
    }{
        {"valid URL", "https://example.com", "abc123", false},
        {"empty URL", "", "", true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Shorten(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Shorten() error = %v, wantErr %v", err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("Shorten() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

### Testing Commands
- Run all tests: `go test ./...`
- With coverage: `go test -cover -coverprofile=coverage.out ./...`
- Race detection: `go test -race ./...`
- Verbose: `go test -v ./...`

## Lambda-Specific Patterns

### Handler Structure
```go
func main() {
    cfg, _ := config.LoadDefaultConfig(context.Background())
    client := dynamodb.NewFromConfig(cfg)
    handler := shortener.NewHandler(client)
    lambda.Start(handler.Handle)
}
```

### Dependencies
- `github.com/aws/aws-lambda-go` for Lambda runtime
- `github.com/aws/aws-sdk-go-v2` for AWS services
```

---

## Skill: Java

### `.agent/skills/java/SKILL.md`

```yaml
---
name: java
description: Java/Spring Boot project scaffolding, patterns, and testing
---

## Project Layout (Spring Boot)

```text
src/
├── main/
│   ├── java/com/company/project/
│   │   ├── Application.java        → Main entry point
│   │   ├── config/                  → Configuration classes
│   │   ├── controller/              → REST controllers
│   │   ├── service/                 → Business logic
│   │   ├── repository/              → Data access layer
│   │   ├── model/                   → Domain entities & DTOs
│   │   └── exception/               → Custom exceptions
│   └── resources/
│       ├── application.yml          → Config
│       └── application-{profile}.yml
└── test/
    └── java/com/company/project/
        ├── controller/              → Controller tests
        ├── service/                 → Service unit tests
        └── integration/             → Integration tests
```

## Standards

### Dependency Injection
- Use constructor injection (not field injection)
- Mark dependencies as `final`
- Use `@RequiredArgsConstructor` (Lombok) or explicit constructors

### REST Controllers
- Use `@RestController` + `@RequestMapping`
- Return `ResponseEntity<T>` for explicit status codes
- Use `@Valid` for request validation
- Handle exceptions with `@ControllerAdvice`

### Testing
- Unit tests: JUnit 5 + Mockito
- Integration tests: `@SpringBootTest` + TestContainers
- API tests: MockMvc or WebTestClient
- Naming: `should_returnShortUrl_when_validUrlProvided()`

### Build
- Maven: `mvn clean verify`
- Gradle: `./gradlew clean build`
```

---

## Skill: JavaScript / TypeScript

### `.agent/skills/javascript/SKILL.md`

```yaml
---
name: javascript
description: JavaScript/TypeScript project setup, React patterns, and Node.js best practices
---

## Project Types

### Node.js API
```text
src/
├── routes/          → Express/Fastify route handlers
├── services/        → Business logic
├── models/          → Data models / schemas
├── middleware/      → Auth, logging, error handling
├── utils/           → Shared utilities
├── config/          → Environment configuration
└── index.ts         → Entry point
```

### React / Next.js Frontend
```text
src/
├── app/             → Next.js app router pages
├── components/      → Reusable UI components
│   ├── ui/          → Primitive components (Button, Input)
│   └── features/    → Feature-specific components
├── hooks/           → Custom React hooks
├── lib/             → Utility functions
├── services/        → API client functions
└── types/           → TypeScript type definitions
```

## Standards

### TypeScript
- Enable `strict` mode in tsconfig.json
- Use explicit return types on exported functions
- Prefer `interface` over `type` for object shapes
- Use `unknown` instead of `any` — if `any` is required, add a comment explaining why

### Error Handling
- Use custom error classes extending `Error`
- Always catch async errors (no unhandled promise rejections)
- Use Result pattern for expected failures: `{ ok: true, data } | { ok: false, error }`

### Testing
- Framework: Vitest (preferred) or Jest
- Component testing: React Testing Library
- E2E: Playwright
- Name tests descriptively: `it('should redirect to original URL when valid code is provided')`

### Package Management
- Use `npm` (not yarn/pnpm) unless project specifies otherwise
- Pin exact versions in production: `--save-exact`
- Keep `package-lock.json` in version control
```

---

## Skill: AWS Infrastructure

### `.agent/skills/aws/SKILL.md`

```yaml
---
name: aws
description: AWS service patterns, security best practices, and cost optimization
---

## Core Principles

### Security
- **Least privilege IAM**: Never use `*` in resource ARNs for production
- **No hardcoded credentials**: Use IAM roles, not access keys
- **Encryption at rest**: Enable on all storage services (S3, DynamoDB, RDS)
- **Encryption in transit**: HTTPS everywhere, TLS 1.2+

### Cost Optimization
- Use **on-demand/pay-per-request** for unpredictable or low workloads
- Use **reserved capacity** only after 3+ months of stable usage data
- Enable **CloudWatch billing alerts** on every account
- Tag all resources: `Project`, `Environment`, `Owner`

### Reliability
- Multi-AZ for production databases
- Dead letter queues for all async processing
- CloudWatch alarms for error rates and latency P99

## Common Service Patterns

### API Gateway + Lambda
- Use **HTTP API** (v2) for REST — cheaper and faster than REST API (v1)
- Set reasonable Lambda timeout (10-30s for APIs, up to 15min for async)
- Configure Lambda memory based on profiling (start at 128MB for Go)
- Use Lambda Powertools for structured logging and tracing

### DynamoDB
- Design access patterns FIRST, then design tables
- Use single-table design when possible
- Enable point-in-time recovery for production
- Use TTL for data with natural expiry (sessions, logs, analytics events)
- Monitor CapacityUnits to right-size provisioned throughput

### S3
- Enable versioning on production buckets
- Set lifecycle policies for cost management
- Block public access by default
- Use presigned URLs for temporary access

## Naming Convention

```text
{project}-{environment}-{service}-{purpose}

Examples:
  url-shortener-prod-dynamodb-urls
  url-shortener-dev-lambda-create
  url-shortener-prod-apigw-api
```
```

---

## Skill: Terraform

### `.agent/skills/terraform/SKILL.md`

```yaml
---
name: terraform
description: Terraform module design, state management, and AWS infrastructure patterns
---

## File Organization

```text
terraform/
├── main.tf           → Primary resources
├── variables.tf      → Input variables with descriptions and validation
├── outputs.tf        → Output values
├── providers.tf      → Provider configuration and version constraints
├── backend.tf        → Remote state configuration
├── locals.tf         → Local values and computed expressions
├── data.tf           → Data sources
└── modules/          → Reusable child modules
    └── lambda/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

## Standards

### Variables
- Always include `description` and `type`
- Use `validation` blocks for input constraints
- Use `sensitive = true` for secrets
- Provide sensible `default` values where appropriate

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### Naming
- Use `snake_case` for all Terraform identifiers
- Prefix resources with project: `url_shortener_lambda_create`
- Use meaningful names, not generic (`this`, `main`, `default`)

### State Management
- Always use remote state (S3 + DynamoDB locking)
- Never commit `.tfstate` files
- Use separate state files per environment
- Enable state encryption

```hcl
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "url-shortener/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### Best Practices
- Pin provider versions: `~> 5.0` (pessimistic constraint)
- Use `terraform fmt` before every commit
- Run `terraform validate` in CI
- Always run `terraform plan` before `terraform apply`
- Use `-target` sparingly — prefer full applies
- Tag all resources with: Project, Environment, ManagedBy=terraform

### Modules
- Extract repeated patterns into modules
- Modules should be small and focused (one concern)
- Use `for_each` over `count` for named resources
- Document modules with README.md
```

---

## Creating Your Own Skills

### Template

```yaml
---
name: your-skill-name
description: One-line description of what this skill covers
---

## When to Use This Skill
[Describe the trigger conditions]

## Standards
[Your team's conventions]

## Patterns
[Code patterns and examples]

## Anti-Patterns
[What NOT to do]

## Commands
[Useful commands for this technology]
```

### Tips for Effective Skills

1. **Be specific** — "Use `slog` for logging" is better than "use structured logging"
2. **Include examples** — Show don't tell. Code snippets > prose
3. **Document anti-patterns** — Tell the agent what NOT to do
4. **Keep skills focused** — One skill per technology/domain
5. **Version control skills** — They evolve with your team's conventions

---

**Previous**: [← Architecture-First Development](03-architecture-first.md) | **Next**: [Example Project →](05-example-project.md)
