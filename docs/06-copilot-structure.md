## The Copilot Equivalent of CLAUDE.md

The closest equivalent is **`.github/copilot-instructions.md`** — a repo-level file Copilot reads automatically. It should contain information about your project such as how to build and test it, and any coding standards or conventions you want Copilot to follow. It also applies to Copilot Chat and code review.

## The Full Folder Structure (3-Layer System)

The community-standard agentic project structure has three layers:

```
PROJECT ROOT
│
├── [LAYER 1: FOUNDATION]
│   ├── .github/copilot-instructions.md   ← VS Code reads this (always-on)
│   └── AGENTS.md                         ← CLI reads this
│
├── [LAYER 2: SPECIALISTS]
│   └── .github/agents/*.agent.md         ← specialist agent personas
│
└── [LAYER 3: CAPABILITIES]
    ├── .github/skills/*.md               ← complex multi-step workflows
    ├── .github/prompts/*.prompt.md       ← quick reusable snippets
    └── .github/instructions/*.instructions.md  ← language/file-specific rules
```


**Layer 1** is your always-active global context (like CLAUDE.md). **Layer 2** are specialist agents like `@docs-agent`, `@test-agent`, `@security-agent`. **Layer 3** are skills that activate on-demand when relevant.

## What to Put in Each File

The six core areas to cover in your agent files are: commands, testing, project structure, code style, git workflow, and boundaries. Be specific about your stack — say "React 18 with TypeScript, Vite, and Tailwind CSS" not just "React project." Include exact versions and key dependencies. One real code snippet showing your style beats three paragraphs describing it.

The `instructions.md` files with `applyTo` scoping let you apply file-specific rules — for example, targeting only `src/components/**/*.tsx` with your React/TypeScript conventions.

## Key Resources

These are the best places to go:

- **[github/awesome-copilot](https://awesome-copilot.github.com)** — the official community repo. It has 175+ agents, 208+ skills, 176+ instructions, 48+ plugins, 7 agentic workflows — all community contributed. You can install directly with `copilot plugin install <name>@awesome-copilot`.

- **[GitHub Blog: How to write a great agents.md](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)** — lessons from analyzing 2,500+ repos, very practical.

- **[GitHub Blog: Maximize Copilot's agentic capabilities](https://github.blog/ai-and-ml/github-copilot/how-to-maximize-github-copilots-agentic-capabilities/)** — focuses on multi-file, architecture-aware workflows.

- **[VS Code Agent Skills docs](https://code.visualstudio.com/docs/copilot/customization/agent-skills)** — official reference for the skills system.

## Quick Start Tip

The first time you ask Copilot to create a pull request in a repo, it will leave a comment with a link to auto-generate custom instructions. You can also ask Copilot to generate the `copilot-instructions.md` for you using the recommended onboarding prompt. It'll scan your repo and write it — much faster than doing it from scratch.