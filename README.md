# How-To: Agentic Coding

> Build real-world projects from scratch using AI coding agents — without writing a single line of code by hand.

This repository covers **two things**:

### Pillar 1: Agent SDLC — The System

The **Agentic Software Development Lifecycle** — the tools, infrastructure, and workflow systems that make AI-assisted development possible. What agents are, how they work, how to configure them.

### Pillar 2: Spec-Driven Development — The Methodology

How leading AI-forward engineering teams actually use the Agent SDLC to **build products from specification documents**. Specs are the source of truth; code is a generated derivative.

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Pillar 1: AGENT SDLC (the system)                               │
│  ┌────────┐ ┌───────┐ ┌────────┐ ┌─────┐ ┌───────────┐          │
│  │ Agents │ │ Tools │ │ Skills │ │ MCP │ │ Workflows │          │
│  └────────┘ └───────┘ └────────┘ └─────┘ └───────────┘          │
│       ↕           ↕          ↕        ↕         ↕                │
│  ┌──────────────────────────────────────────────────────┐        │
│  │  Pillar 2: SPEC-DRIVEN DEVELOPMENT (the methodology) │        │
│  │                                                      │        │
│  │  Requirements Spec → API Spec → Architecture Spec    │        │
│  │       ↓                  ↓              ↓            │        │
│  │  Agent generates: Code, Tests, Infra, Docs           │        │
│  └──────────────────────────────────────────────────────┘        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Documentation

### Pillar 1: Agent SDLC

| Doc | Topic |
|-----|-------|
| [Core Concepts](docs/01-core-concepts.md) | Agents, Tools, Skills, MCP Servers, RAG — what they are and how they connect |
| [Project Setup](docs/02-project-setup.md) | Configure VS Code, agent instructions, MCP servers, skills, and workflows |
| [Skills Catalog](docs/04-skills-catalog.md) | Reusable skill folders for Go, Java, JS, AWS, Terraform |

### Pillar 2: Spec-Driven Development

| Doc | Topic |
|-----|-------|
| [Architecture-First Development](docs/03-architecture-first.md) | The "zero-code-first" methodology — requirements, ADRs, AWS planning |
| [Example Project](docs/05-example-project.md) | End-to-end URL Shortener built from spec documents through agent conversations |

---

## Example Project

The `examples/url-shortener/` directory is a complete **spec-driven project** — a URL Shortener on AWS (API Gateway → Lambda → DynamoDB) where specs are the source of truth and everything else is agent-generated.

```
examples/url-shortener/
├── specs/                   ← SOURCE OF TRUTH (human-written)
│   ├── requirements.md
│   ├── api-spec.yaml
│   ├── architecture.md
│   └── data-model.md
├── .agent/                  ← Agent configs, workflows
├── AGENTS.md / CLAUDE.md    ← Agent instructions
└── (code, infra, tests)     ← GENERATED FROM SPECS
```

---

## Who Is This For?

- Developers exploring AI-assisted development for the first time
- Teams evaluating Agent SDLC workflows for production use
- Architects who want to lead with specs, not boilerplate
- Anyone curious about MCP, RAG, and the agent ecosystem

## Quick Start

1. Read [Core Concepts](docs/01-core-concepts.md) to understand the Agent SDLC vocabulary
2. Follow [Project Setup](docs/02-project-setup.md) to configure your environment
3. Read [Architecture-First](docs/03-architecture-first.md) to learn spec-driven methodology
4. Study the [Example Project](docs/05-example-project.md) to see the full workflow in action

## License

MIT
